---
layout: post
title: Creating CooperDB
---
Yesterday I [introduced CooperDB]({% post_url 2016-04-01-introducing-cooperdb %}), the first distributed, share-nothing database that is fully partition tolerant, highly available, **and** immediately consistent.  I promised to follow up with a post on the creation of CooperDB - which is what you are reading now.

### CooperDB API
CooperDB has a very straightforward REST interface.  It supports two methods:

* POST &lt;path&gt; - Store any payload of any content type at the specified path.  Returns HTTP status code 201 - Created.
* GET &lt;path&gt; - Retrieve the payload previously stored at the specified path.  Returns HTTP status code 404 - Not Found.

Remember, one of the unique features of CooperDB is that you can send whatever you want to it to store it, and then you will never be able to find it again.  This key feature is what makes it possible to be partition tolerant, highly available, and immediately consistent all at the same time.

### Writing CooperDB
[Zoomulus Servers](https://github.com/zoomulus/servers) seems a great place to start, since it was designed to make it super easy to create a simple web service as quickly as possible.  It turned out to be quite straightforward.  Here are the steps:

#### Add com.zoomulus.servers
I created a new Maven project, and added `com.zoomulus.servers` as a dependency:

{% highlight xml %}
<dependency>
  <groupId>com.zoomulus</groupId>
  <artifactId>servers</artifactId>
  <version>0.0.2</version>
</dependency>
{% endhighlight %}

At the time of this writing, version 0.0.2 of `com.zoomulus.servers` is the latest available on Sonatype so I'll use that for now.

#### Create the server
CooperDB is essentially an HTTP web server as defined in `com.zoomulus.servers`.  Creating the server is as simple as extending `com.zoomulus.servers.http.HttpServer`:

{% highlight java %}
public final class CooperDBServer extends HttpServer
{
  @Inject
  public CooperDBServer(final Injector injector,
                        @Named(LISTEN_PORT_NAME) final Integer port)
  {
    super(
      HttpServerConnector.builder()
        .port(port.intValue())
        .injector(injector)
        .build()
    );
  }
    
  public static void main(String[] args)
  {
    Guice.createInjector(new CooperDBServerModule())
      .getInstance(CooperDBServer.class)
      .start();
  }
}
{% endhighlight %}

Using Guice to create our server class allows us to inject the port number and the `HttpResponder` subclass which will handle the requests.  Zoomulus Servers expects this and leverages it to make creating a server really easy.  The `CooperDBServer` constructor simply passes the port and the injector along to the library underneath.

#### Define the Guice module
We used the class `CooperDBServerModule` as the Guice module for our dependency injection in the `main` function of `CooperDBServer` above.  That module needs to define the port number we will listen on and our `HttpResponder` subclass to use:

{% highlight java %}
public class CooperDBServerModule extends AbstractModule
{
    @Override
    protected void configure()
    {
        bind(HttpResponder.class)
          .to(CooperDBServerResponder.class);
        bind(Integer.class)
          .annotatedWith(Names.named(HttpServer.LISTEN_PORT_NAME))
          .toInstance(22305);
    }
}
{% endhighlight %}

#### Define the HttpResponder
The module bound `HttpResponder` to the class `CooperDBServerResponder` which is the last thing we need to define.  Remember the requirements:

* It can accept a POST on any path and respond with 201 - Created.
* It can accept a GET on any previously POSTed path and respond with 404 - Not Found.
* Any other method is unsupported.

Of course, if a GET is received on a path that **was not** previously posted, the answer would also be 404 - Not Found, which makes this quite a bit easier.

Here's the responder:

{% highlight java %}
public class CooperDBServerResponder extends HttpResponder
{

    @Override
    protected boolean haveMatchingResource(final HttpRequest request)
    {
        return true;
    }

    @Override
    protected boolean isSupportedMethod(final HttpRequest request)
    {
        final HttpMethod method = request.getMethod();
        return method == HttpMethod.GET || method == HttpMethod.POST;
    }

    @Override
    protected FullHttpResponse generateResponse(final HttpRequest request,
                                                final HttpHeaders headers)
    {
        final HttpMethod method = request.getMethod();
        if (method == HttpMethod.GET)
        {
            return createResponse(HttpResponseStatus.NOT_FOUND);
        }
        else if (method == HttpMethod.POST)
        {
            return createResponse(HttpResponseStatus.CREATED);
        }
        else
        {
            return createResponse(HttpResponseStatus.METHOD_NOT_ALLOWED);
        }
    }
}
{% endhighlight %}

There are three methods we have to implement from `HttpResponder`.

* `haveMatchingResource` basically tells the library "I know what to do with this request resource" (or path).  Since we know how to handle everything, we return `true` here.  Because the library will automatically generate a 404 if `haveMatchingResource` returns `false`, we could also have inspected the request and returned `false` here if it was a GET method.  Either approach would work.
* `isSupportedMethod` tells the library "I know how to handle this request method".  We support two methods - GET and POST.
* `generateResponse` actually puts the response together that is sent back from the server.  This is easy:
  * If it is a GET, return 404
  * If it is a POST, return 201
  * If it is anything else (which should not happen), return 405
  
That's all there is to it.

### In Seriousness...
CooperDB is a fun joke, but it does do one important thing:  It shows just how easy it is to create a functioning web service using the Zoomulus Servers library.  Over time I'll use this to evolve them both into something even more easy to use.

CooperDB is available [on GitHub](https://github.com/mattvryan/cooperdb).