# How JavaScript Works: Inside the Networking Layer + How to Optimize Its Performance and Security


Alexander Zlatkov


Alexander Zlatkov


Following


Apr 12, 2018 · 12 min read


[How JavaScript Works: Inside the Networking Layer + How to Optimize Its Performance and Security](https://blog.sessionstack.com/how-javascript-works-inside-the-networking-layer-how-to-optimize-its-performance-and-security-f71b7414d34c)


This is post # 12 of the series dedicated to exploring JavaScript and its building components. In the process of identifying and describing the core elements, we also share some rules of thumb we use when building SessionStack, a JavaScript application that needs to be robust and highly-performant to help users see and reproduce their web app defects real-time.


If you missed the previous chapters, you can find them here:


An overview of the engine, the runtime, and the call stack


Inside Google’s V8 engine + 5 tips on how to write optimized code


Memory management + how to handle 4 common memory leaks


The event loop and the rise of Async programming + 5 ways to better coding with async/await


Deep dive into WebSockets and HTTP/2 with SSE + how to pick the right path


A comparison with WebAssembly + why in certain cases it’s better to use it over JavaScript


The building blocks of Web Workers + 5 cases when you should use them


Service Workers, their life-cycle, and use cases


The mechanics of Web Push Notifications


Tracking changes in the DOM using MutationObserver


The rendering engine and tips to optimize its performance


Just like we mentioned in our previous blog post about the rendering engine, we believe that the difference between good and great JavaScript developers is that the latter not only understand the nuts and bolts of the language but also its internals and the surrounding environment.


A little bit of history


Forty-nine years ago, a thing called ARPAnet was created. It was an early packet switching network and the first network to implement the TCP/IP suite. That network set up a link between University of California and Stanford Research Institute. 20 years later Tim Berners-Lee circulated a proposal for a「Mesh」which later became better known as the World Wide Web. In those 49, years the internet has come a long way, starting from just two computers exchanging packets of data, to reaching more than 75 million servers, 3.8B people using the internet and 1.3B websites.


In this post, we’ll try to analyze what techniques modern browsers employ to automatically boost performance (without you even knowing it), and we’ll specifically zoom in on the browser networking layer. At the end, we’ll provide some ideas on how to help browsers boost even more the performance of your web apps.


Overview


The modern web browser has been specifically designed for the fast, efficient and secure delivery of web apps/websites. With hundreds of components running on different layers, from process management and security sandboxing to GPU pipelines, audio and video, and many more, the web browser looks more like an operating system rather than just a software application.


The overall performance of the browser is determined by a number of large components: parsing, layout, style calculation, JavaScript and WebAssembly execution, rendering, and of course, the networking stack.


Engineers often think that the networking stack is a bottleneck. This is frequently the case since all resources need to be fetched from the internet before the rest of the steps are unblocked. For the networking layer to be efficient it needs to play the role of more than just a simple socket manager. It is presented to us as a very simple mechanism for resource fetching but it’s actually an entire platform with its own optimization criteria, APIs, and services.


As web developers, we don’t have to worry about the individual TCP or UDP packets, request formatting, caching and everything else that’s going on. The entire complexity is taken care of by the browser so we can focus on the application we’re developing. Knowing what’s happening under the hood, however, can help us create faster and more secure applications.


In essence, here’s what’s happening when the user starts interacting with the browser:


The user enters a URL in the browser address bar


Given the URL of a resource on the web, the browser starts by checking its local and application caches and tries to use a local copy to fulfill the request.


If the cache cannot be used, the browser takes the domain name from the URL and requests the IP address of the server from a DNS. If the domain is cached, no DNS query is needed.


The browser creates an HTTP packet saying that it requests a web page located on the remote server.


The packet is sent to the TCP layer which adds its own information on top of the HTTP packet. This information is required to maintain the started session.


The packet is then handed to the IP layer which main job is to figure out a way to send the packet from the user to the remote server. This information is also stored on top of the packet.


The packet is sent to the remote server.


Once the packet is received, the response gets sent back in a similar manner.


The W3C Navigation Timing specification provides a browser API as well as visibility into the timing and performance data behind the life of every request in the browser. Let’s inspect the components, as each plays a critical role in delivering the optimal user experience:


The whole networking process is very complex and there are many different layers which can become a bottleneck. This is why browsers strive to improve performance on their side by using various techniques so that the impact of the entire network communication is minimal.


Socket management


Let’s start with some terminology:


Origin — A triple of application protocol, domain name and port number (e.g. https, www.example.com, 443)


Socket pool — a group of sockets belonging to the same origin (all major browsers limit the maximum pool size to 6 sockets)


JavaScript and WebAssembly do not allow us to manage the lifecycle of individual network sockets, and that’s a good thing! This not only keeps our hands clean but it also allows the browser to automate a lot of performance optimizations some of which include socket reuse, request prioritization and late binding, protocol negotiation, enforcing connection limits, and many other.


Actually, modern browsers go the extra mile to separate the request management cycle from socket management. Sockets are organized in pools, which are grouped by origin, and each pool enforces its own connection limits and security constraints. Pending requests are queued, prioritized, and then bound to individual sockets in the pool. Unless the server intentionally closes the connection, the same socket can be automatically reused across multiple requests!


Since the opening of a new TCP connection comes at an additional cost, the reuse of connections introduces great performance benefits on its own. By default, browsers use the so-called「keepalive」mechanism which saves time from opening a new connection to the server when a request is made. The average time for opening a new TCP connection is:


Local requests — 23ms


Transcontinental requests — 120ms


Intercontinental requests — 225ms


This architecture opens the door to a number of other optimization opportunities. The requests can be executed in a different order depending on their priority. The browser can optimize the bandwidth allocation across all sockets or it can open sockets in anticipation of a request.


As I mentioned before, this is all managed by the browser and does not require any work on our side. But this doesn’t necessarily mean that we can’t do anything to help. Choosing the right network communication patterns, type, and frequency of transfers, choice of protocols and tuning/optimization of our server stack can play a great role in improving the overall performance of an application.


Some browsers even go one step further. For example, Chrome can self-teach itself to get faster as you use it. It learns based on the sites visited and the typical browsing patterns so it can anticipate likely user behavior and take action before the user does anything. The simplest example is pre-rendering a page when the user hovers on a link. If you’re interested in learning more about Chrome’s optimizations, you can check out this chapter https://www.igvita.com/posa/high-performance-networking-in-google-chrome/ of the High-Performance Browser Networking book.


Network Security and Sandboxing


Allowing the browser to manage the individual sockets has another very important purpose: this way the browser enables the enforcement of a consistent set of security and policy constraints on untrusted application resources. For example, the browser does not allow direct API access to raw network sockets as this would enable any malicious application to make arbitrary connections to any host. The browser also enforces connection limits which protect the server as well as the client from resource exhaustion.


The browser formats all outgoing requests to enforce consistent and well-formed protocol semantics to protect the server. Similarly, response decoding is done automatically to protect the user from malicious servers.


TLS negotiation


Transport Layer Security (TLS) is a cryptographic protocol that provides communication security over a computer network. It finds widespread use in many applications, one of which is web browsing. Websites can use TLS to secure all communications between their servers and web browsers.


The entire TLS handshake consists of the following steps:


The client sends a「Client hello」message to the server, along with the client’s random value and supported cipher suites.


The server responds by sending a「Server hello」message to the client, along with the server’s random value.


The server sends its certificate for authentication to the client and may request a similar certificate from the client. The server sends the「Server hello done」message.


If the server has requested a certificate from the client, the client sends it.


The client creates a random Pre-Master Secret and encrypts it with the public key from the server’s certificate, sending the encrypted Pre-Master Secret to the server.


The server receives the Pre-Master Secret. The server and the client each generate the Master Secret and session keys based on the Pre-Master Secret.


The client sends a「Change cipher spec」notification to the server to indicate that the client will start using the new session keys for hashing and encrypting messages. The client also sends a「Client finished」message.


The server receives the「Change cipher spec」and switches its record layer security state to symmetric encryption using the session keys. The server sends a「Server finished」message to the client.


Client and server can now exchange application data over the secured channel they have established. All messages sent from the client to the server and back are encrypted using the session key.


The user is warned in case any of the verifications fail — e.g., the server is using a self-signed certificate.


Same-origin policy


Two pages have the same origin if the protocol, port (if one is specified), and host are the same for both pages.


Here are some examples of resources which may be embedded cross-origin:


JavaScript with <script src=」…」></script>. Error messages for syntax errors are only available for same-origin scripts


CSS with <link rel=」stylesheet」href=」…」>. Due to the relaxed syntax rules of CSS, cross-origin CSS requires a correct Content-Type header. Restrictions vary by browser


Images with <img>


Media files with <video> and <audio>


Plug-ins with <object>, <embed> and <applet>


Fonts with @font-face. Some browsers allow cross-origin fonts, others require same-origin fonts.


Anything with <frame> and <iframe>. A site can use the X-Frame-Options header to prevent this form of cross-origin interaction.


The above list is far from complete; its goal is to highlight the principle of「least privilege」at work. The browser exposes only the APIs and resources that are necessary for the application code: the application supplies the data and the URL, and the browser formats the request and handles the full lifecycle of each connection.


It’s worth noting that there is no single concept of the「same-origin policy.」Instead, there is a set of related mechanisms that enforce restrictions on DOM access, cookie and session state management, networking, and other components of the browser.


Resource and Client State Caching


The best and fastest request is a request not made. Prior to dispatching a request, the browser automatically checks its resource cache, performs the necessary validation checks, and returns a local copy of the resources if the specified conditions are met. If a local resource is not available in the cache, then a network request is made and the response is automatically placed in the cache for a subsequent access if such is permitted.


The browser automatically evaluates caching directives on each resource


The browser automatically revalidates expired resources when possible


The browser automatically manages the size of the cache and resource eviction


Managing an efficient and optimized resource cache is hard. Thankfully, the browser takes care of the entire complexity on our behalf, and all we need to do is ensure that our servers are returning the appropriate cache directives; to learn more, see Cache Resources on the Client. You do provide Cache-Control, ETag, and Last-Modified response headers for all the resources on your pages, right?


Finally, an often-overlooked but critical function of the browser is its duty to provide authentication, session, and cookie management. The browser maintains separate「cookie jars」for each origin, provides necessary application and server APIs to read and write new cookie, session, and authentication data, and automatically appends and processes appropriate HTTP headers to automate the entire process on our behalf.


An example:


A simple but illustrative example of the convenience of deferring session state management to the browser: an authenticated session can be shared across multiple tabs or browser windows, and vice versa; a sign-out action in a single tab will invalidate open sessions in all other open windows.


Application APIs and Protocols


Walking up the ladder of provided network services we finally arrive at the application APIs and protocols. As we saw, the lower layers provide a wide array of critical services: socket and connection management, request and response processing, enforcement of various security policies, caching, and much more. Every time we initiate an HTTP or an XMLHttpRequest, a long-lived Server-Sent Event or WebSocket session, or open a WebRTC connection, we are interacting with some or all of these underlying services.


There is no single best protocol or API. Every non-trivial application will require a mix of different transports based on a variety of requirements: interaction with the browser cache, protocol overhead, message latency, reliability, type of data transfer, and more. Some protocols may offer low-latency delivery (e.g., Server-Sent Events, WebSocket), but may not meet other critical criteria, such as the ability to leverage the browser cache or support efficient binary transfers in all cases.


A Few Things You Can Do To Improve Your Web App Performance and Security


Always use the「Connection: Keep-Alive」header in your requests. Browsers do this by default. Make sure that the server uses the same mechanism.


Use the proper Cache-Control, Etag and Last-Modified headers so you can save the browser some downloading time.


Spend time tweaking and optimizing your web server. This is where real magic can happen! Bear in mind that the process if is very specific for each web app and the type of data you’re transmitting.


Always use TLS! Especially if you have any type of authentication in your application.


Research what security policies browsers provide and enforce them in your application.


Both performance and security are first-class citizens in SessionStack. The reason why we can’t afford to compromise on either is that once SessionStack is integrated into your web app, it starts monitoring everything from DOM changes and user interaction to network requests, unhandled exceptions and debug messages. All the data is transmitted to our servers real-time which allows you to replay issues from your web apps as videos and see everything that happened to your users. And this all is taking place with minimum latency and no performance overhead to your app.


This is why we strive to employ all the above tips + a few more which we’ll discuss in a future post.


There is a free plan if you’d like to give SessionStack a try.


Resources


https://hpbn.co/


https://www.amazon.com/Tangled-Web-Securing-Modern-Applications/dp/1593273886


https://msdn.microsoft.com/en-us/library/windows/desktop/aa380513(v=vs.85).aspx


http://www.internetlivestats.com/


http://vanseodesign.com/web-design/browser-requests/


