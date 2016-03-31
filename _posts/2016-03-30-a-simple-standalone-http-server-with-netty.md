---
layout: post
title: A Simple Standalone HTTP Server with Netty
---
When creating a software system, regardless of size, composability is a key feature.  This applies to functions and classes, and in distributed systems it applies to microservices as well.  A programming language encourages composability by making the units of composition easy to create.  Thus, microservices should be easy to create also.  And, in some languages, they are.

This isn't always true in Java, however.  Java's robust ecosystem makes me want to use it for developing distributed systems, but it simply takes way too long to go from zero lines of code to a functional REST service.

Hence the non-creatively-named "servers" component of my [Zoomulus](https://github.com/zoomulus) project.  The goals are:

* Build atop `java.nio` for performance
* Create a generic and flexible base-level implementation for future uses
* Abstract as many details away from the consumer as possible, so you focus more on business logic and less on setup
* Make it easy to create a stand-alone service quickly

### A Simple Netty HTTP Server
Netty was an obvious choice to get started here.  It's built atop java.nio and is really powerful and flexible.  It also requires a bit of study to know how to get it set up.  Here's an example of a fairly "simple" HTTP server with Netty.  (You can download the full example from [my gist](https://gist.github.com/mattvryan/6fd31c2b2e4757da3116ec247ab43f0b).)

{% highlight java %}
public class NettyHttpServer
{
  private ChannelFuture channel;
  private final EventLoopGroup masterGroup;
  private final EventLoopGroup slaveGroup;
    
  public NettyHttpServer()
  {
    masterGroup = new NioEventLoopGroup();
    slaveGroup = new NioEventLoopGroup();        
  }

  public void start() // #1
  {
    Runtime.getRuntime().addShutdownHook(new Thread()
    {
      @Override
      public void run() { shutdown(); }
    });
        
    try
    {
      // #3
      final ServerBootstrap bootstrap =
        new ServerBootstrap()
          .group(masterGroup, slaveGroup)
          .channel(NioServerSocketChannel.class)
          .childHandler(new ChannelInitializer<SocketChannel>() // #4
          {
            @Override
            public void initChannel(final SocketChannel ch)
              throws Exception
            {
              ch.pipeline().addLast("codec", new HttpServerCodec());
              ch.pipeline().addLast("aggregator",
                new HttpObjectAggregator(512*1024));
              ch.pipeline().addLast("request",
                new ChannelInboundHandlerAdapter() // #5
              {
                @Override
                public void channelRead(ChannelHandlerContext ctx, Object msg)
                  throws Exception
                {
                  if (msg instanceof FullHttpRequest)
                  {
                    final FullHttpRequest request = (FullHttpRequest) msg;
                                            
                    final String responseMessage = "Hello from Netty!";
                                            
                    FullHttpResponse response = new DefaultFullHttpResponse(
                      HttpVersion.HTTP_1_1,
                      HttpResponseStatus.OK,
                      copiedBuffer(responseMessage.getBytes())
                    );
    
                    if (HttpHeaders.isKeepAlive(request))
                    {
                      response.headers().set(
                        HttpHeaders.Names.CONNECTION,
                        HttpHeaders.Values.KEEP_ALIVE
                      );
                    }
                    response.headers().set(HttpHeaders.Names.CONTENT_TYPE,
                      "text/plain");
                    response.headers().set(HttpHeaders.Names.CONTENT_LENGTH,
                      responseMessage.length());
                                        
                    ctx.writeAndFlush(response);
                  }
                  else
                  {
                    super.channelRead(ctx, msg);
                  }
                }
    
                @Override
                public void channelReadComplete(ChannelHandlerContext ctx)
                  throws Exception
                {
                  ctx.flush();
                }
    
                @Override
                public void exceptionCaught(ChannelHandlerContext ctx,
                  Throwable cause) throws Exception
                {
                  ctx.writeAndFlush(new DefaultFullHttpResponse(
                    HttpVersion.HTTP_1_1,
                    HttpResponseStatus.INTERNAL_SERVER_ERROR,
                    copiedBuffer(cause.getMessage().getBytes())
                  ));
                }                                    
              });
            }
          })
          .option(ChannelOption.SO_BACKLOG, 128)
          .childOption(ChannelOption.SO_KEEPALIVE, true);
      channel = bootstrap.bind(8080).sync();
    }
    catch (final InterruptedException e) { }
  }
    
  public void shutdown() // #2
  {
    slaveGroup.shutdownGracefully();
    masterGroup.shutdownGracefully();

    try
    {
      channel.channel().closeFuture().sync();
    }
    catch (InterruptedException e) { }
  }

  public static void main(String[] args)
  {
    new NettyHttpServer().start();
  }
}
{% endhighlight %}

You're right - this doesn't look "simple" at all.  It turns out setting up something that seems fairly basic, even with a great foundation like Netty, is more complicated than it feels like it should be.  You're trying to solve a business problem, not write a server.

Of course, we would never actually write our server like this, putting everything into a single function in a single class.  I wrote the example this way to help answer this question:  What does it take to create a Netty HTTP server anyway?

First, you need it to act like a server.  In our case, this is essentially done with the `start()` (#1) and `shutdown()` (#2) methods.  When the application runs, we invoke the `start()` method.  One of the first things it does is set up a shutdown hook to call `shutdown()`.  That way a simple CTRL+C or SIGTERM can cause the server to gracefully exit.  No matter how many servers we create, we will probably want code like this for all of them.

Second, we have to bootstrap the server.  In this example this is done within the `try` block in the `start()` method (#3).  In Netty, bootstrapping the server consists of building a `ServerBootstrap` object and then keeping track of the `ChannelFuture` it creates.  When the server shuts down, we will close that `ChannelFuture`.  Again, no matter how many servers we create, we will probably want code like this for all of them.  Really the only things that might vary from one HTTP server to another would be the port we are listening on and the child handlers we register with the `ServerBootstrap`.

Third, we have to add a child handler.  In this example we defined the child handler inline and added it to the `ServerBootstrap` via its `childHandler()` method (#4).  The `initHandler()` method of the HTTP child handler sets up a request handling pipeline, running it through a number of Netty-provided handlers before it gets handed off to our own custom handler.  Again, this step would probably be the same for most HTTP servers.

Finally, we add our own custom handler (#5).  This handles the details of the HTTP request for our particular implementation.  But even here, there's a lot of common things to handle, including a lot that we didn't take care of in this example, like whether the request has permission to perform the requested action, is the requested method supported, is the protocol version supported, is the requested path found, etc.  Most of this could be handled for us also.

To really accomplish my goals, I would need to make this a lot simpler.

### The Zoomulus HTTP Server

#### Beginning at the End
When I started out, I wrote a test server first - an example of how a consumer would consume this library.  I wanted to know where I was heading.

It didn't start out exactly like this, but was pretty close.

{% highlight java %}
public class TestHttpServerRunner extends HttpServer
{
  final Injector injector;
    
  @Inject
  public TestHttpServerRunner(final Injector injector)
  {
    this.injector = injector;
  }
    
  public static void main(final String[] args)
  {
    Guice.createInjector(new TestHttpServerModule())
      .getInstance(TestHttpServerRunner.class)
      .start();
  }
}
{% endhighlight %}

Using Guice for dependency injection offers consumers a lot of flexibility at the implementation level.  The dependency injection is defined in the class `TestHttpServerModule`.  This post isn't about dependency injection or Guice here so I won't go into details.  Suffice it to say that in the simple case it only defines two injectable things:  The port I want to listen to, and an abstract class implementation that will actually generate the responses.

#### Defining the Server
`HttpServer` is a class in the `com.zoomulus.servers` project:

{% highlight java %}
public class HttpServer extends BasicNettyServer
{
  public static final String LISTEN_PORT_NAME = "listenPort";
    
  @Inject
  public HttpServer(final Injector injector,
                    @Named(LISTEN_PORT_NAME) int port)
  {
    super(HttpServerConnector.builder()
      .port(port)
      .injector(injector)
      .build());
  }
    
  @Inject
  public HttpServer(final ServerConnector connector)
  {
    super(connector);
  }
}
{% endhighlight %}

Notice how it accepts the `@Named` parameter `LISTEN_PORT_NAME`.  This is the port defined in our injection module which tells us the port we will listen on.

`HttpServer` itself extends the class `BasicNettyServer`, which is where a lot of the fun happens:

{% highlight java %}
public class BasicNettyServer implements Server
{
  private final List<ServerConnector> connectors;
  private List<ChannelFuture> channels = Lists.newArrayList();
  private final EventLoopGroup masterGroup;
  private final EventLoopGroup slaveGroup;
    
  public BasicNettyServer(final ServerConnector connector)
  {
    this(Lists.newArrayList(connector));
  }

  public BasicNettyServer(final List<ServerConnector> connectors)
  {
    this.connectors = connectors;
    masterGroup = new NioEventLoopGroup();
    slaveGroup = new NioEventLoopGroup();
  }
    
  public void start()
  {
    Runtime.getRuntime().addShutdownHook(new Thread()
    {
      @Override
      public void run() { shutdown(); }
    });

    try
    {
      // for each connector, build a bootstrap,
      // start and save the ChannelFuture
      for (final ServerConnector connector : connectors)
      {
        final ServerBootstrap bootstrap =
          new ServerBootstrap()
            .group(masterGroup, slaveGroup)
            .channel(NioServerSocketChannel.class)
            .childHandler(connector.getChannelInitializer())
            .option(ChannelOption.SO_BACKLOG, 128)
            .childOption(ChannelOption.SO_KEEPALIVE, true);
          channels.add(bootstrap.bind(connector.getPort()).sync());
      }
    }
    catch (final InterruptedException e) { }
  }

  public void shutdown()
  {
    slaveGroup.shutdownGracefully();
    masterGroup.shutdownGracefully();

    for (final ChannelFuture channel : channels)
    {
      try
      {
        channel.channel().closeFuture().sync();
      }
      catch (InterruptedException e) { }
    }
  }
}
{% endhighlight %}

The `BasicNettyServer` class takes care of two of the things we discussed earlier.  First, it sets up the basic server `start()` and `shutdown()` methods, and second, it takes care of bootstrapping the server.  In this case, we support any number of `ServerConnector` objects and create a bootstrap for each one, managing the lifecycle of them all.  This allows a single server to listen on multiple ports if we so choose.

`ServerConnector` is part of `com.zoomulus.servers`; we will look at that next.

#### Setting Up the ChannelInitializer
The `ServerConnector` interface is used to contain implementation details needed to bootstrap the Netty server.  `com.zoomulus.servers` includes a default implementation called `HttpServerConnector` which is usually sufficient.

{% highlight java %}
public class HttpServerConnector implements ServerConnector
{
  int port;
  final Optional<Injector> injector;
    
  public static final String CODEC_HANDLER_NAME  = "codec_handler";
  public static final String COMPRESSOR_HANDLER_NAME = "compressor_handler";
  public static final String AGGREGATOR_HANDLER_NAME = "aggregator_handler";
  public static final String HTTP_REQUEST_HANDLER_NAME = "http_request_handler";

  @Override
  public ChannelInitializer<?> getChannelInitializer()
  {
    return new ChannelInitializer<SocketChannel>()
    {
      @Override
      public void initChannel(final SocketChannel ch) throws Exception
      {
        ch.pipeline().addLast(CODEC_HANDLER_NAME, new HttpServerCodec());
        ch.pipeline().addLast(AGGREGATOR_HANDLER_NAME,
          new HttpObjectAggregator(512*1024));
        ch.pipeline().addLast(HTTP_REQUEST_HANDLER_NAME,
          injector.get().getInstance(HttpHandler.class));
                
        if (compress())
        {
          ch.pipeline().addAfter(CODEC_HANDLER_NAME,
            COMPRESSOR_HANDLER_NAME,
            new HttpContentCompressor()
          );
        }
      }
    };
  }
  ...
{% endhighlight %}

This snippet isn't all of `HttpServerConnector` but shows the main part - how the Netty `ChannelInitializer` gets set up, which was the third item in our list of things our simple server needed to handle.  Using our `injector` to obtain an `HttpHandler` instance allows us to inject an `HttpResponder` which is where we will implement the custom details of how this HTTP server responds to requests.

`HttpHandler` and `HttpResponder` are both part of `com.zoomulus.servers`.  Let's take a look at how those finish the picture.

#### Defining our Responder
The last thing to do to get the server set up is to define a responder.  Responders extend `HttpResponder` which does most of the work for you.  Here's a very simple responder that always answers with a static message:

{% highlight java %}
public final class DefaultHttpResponder extends HttpResponder
{
  @Override
  protected boolean haveMatchingResource(HttpRequest request)
  {
    return true;
  }

  @Override
  protected boolean isSupportedMethod(HttpRequest request)
  {
    return true;
  }

  @Override
  protected FullHttpResponse generateResponse(HttpRequest request,
                                              HttpHeaders headers)
  {
    return createSuccessResponse("Hello from Netty!");
  }
}
{% endhighlight %}

`HttpResponder` and `HttpHandler` take care of the bulk of the HTTP processing we had left to do in our final step in the "simple" server above, meaning all our custom responder has to do is generate the response text.

Now we can simply run the server at the command line and test it with a simple tool like `curl`.

### Creating a Custom HTTP Server
Now that we have this framework in place, creating our own custom HTTP server should be as simple as:

* Define our own `HttpResponder` subclass that does our custom response handling
* Define our own Guice module that sets up our `HttpResponder` subclass for injection
* Define our own `HttpServer` subclass that constructs the server using our module for dependency injection

That's it.  In a future post I'll show how easy this is to do.

`com.zoomulus.servers` is available [on GitHub](https://github.com/zoomulus/servers) and [on Sonatype](https://oss.sonatype.org/) so you can include it in your project with Maven.

