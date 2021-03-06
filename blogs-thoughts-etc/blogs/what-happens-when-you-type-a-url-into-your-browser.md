# What happens when you type a URL into your browser?

### **Action**

Bob enters a URL into the browser and hits Enter. In this example, the URL is composed of 4 parts:       **๐๐๐๐://๐๐๐๐๐๐๐.๐๐๐/๐๐๐๐๐๐๐/๐๐๐๐๐๐๐๐/๐๐๐๐๐**

&#x20;           ๐น scheme - ๐๐๐๐://. This tells the browser to send a connection to the server using HTTPS.&#x20;

&#x20;           ๐น domain - ๐๐๐๐๐๐๐.๐๐๐. This is the domain name of the site.

&#x20;           ๐น path - ๐๐๐๐๐๐๐/๐๐๐๐๐๐๐๐. It is the path on the server to the requested resource: phone.&#x20;

&#x20;           ๐น resource - ๐๐๐๐๐. It is the name of the resource Bob wants to visit.



### DNS LOOKUP  <a href="#dns-lookup" id="dns-lookup"></a>

First, the URL, or domain name, must be converted into an IP address that the browser can use to send an HTTP request. Each domain name is associated with an IP address.

The browser looks up the IP address for the domain with a domain name system (DNS) lookup. To make the lookup process fast, data is cached at different layers: browser cache, OS cache, local network cache, and ISP cache.

&#x20;If the IP address cannot be found at any of the caches, the browser goes to DNS servers to do a **recursive DNS lookup** until the IP address is found .



### HTTP REQUEST <a href="#http-request" id="http-request"></a>

Now that we have the IP address of the server, the browser establishes a TCP connection with the server.

The browser sends an HTTP request to the server. The request looks like this:

&#x20; `๐๐๐ /๐ฑ๐ฉ๐ฐ๐ฏ๐ฆ ๐๐๐๐/1.1 ๐๐ฐ๐ด๐ต: ๐ฆ๐น๐ข๐ฎ๐ฑ๐ญ๐ฆ.๐ค๐ฐ๐ฎ`

The HTTP request must go through many networking layers (for example SSL if itโs encrypted). These layers generally serve to protect the integrity of the data and do error correction.&#x20;

_For example, the TCP layer handles reliability of the data and orderedness. If packets underneath the TCP layer are corrupted (detected via checksum), the protocol dictates that the request must be resent. If packets arrive in the wrong order, it will reorder them._

But basically, in the end, the server will receive a request from the client at the URL specified, along with metadata in the headers and cookies.



### SERVER HANDLER <a href="#server-handler" id="server-handler"></a>

Now the request has been received by some server. Popular server engines are nginx and Apache. What these servers do is handle it accordingly. If the website is static, for example, the server can server a file from the local file system. More often, though, the request is forwarded to an application running on a web framework such as Django or Ruby on Rails.

What these applications do is eventually return a response to the request, but sometimes it may have to perform some logic to serve it. For example, if youโre on Facebook, their servers will parse the request, see that you are logged in, and query their databases and get the data for your Facebook Feed.

Fundamentally the server processes the request and sends back the response. For a successful response (the status code is 200). The HTML response might look like this:

`๐๐๐๐/1.1 200 ๐๐ ๐๐ข๐ต๐ฆ: ๐๐ถ๐ฏ, 30 ๐๐ข๐ฏ 2022 00:01:01 ๐๐๐ ๐๐ฆ๐ณ๐ท๐ฆ๐ณ: ๐๐ฑ๐ข๐ค๐ฉ๐ฆ ๐๐ฐ๐ฏ๐ต๐ฆ๐ฏ๐ต-๐๐บ๐ฑ๐ฆ: ๐ต๐ฆ๐น๐ต/๐ฉ๐ต๐ฎ๐ญ; ๐ค๐ฉ๐ข๐ณ๐ด๐ฆ๐ต=๐ถ๐ต๐ง-8`

`<!๐๐๐๐๐?๐๐ ๐ฉ๐ต๐ฎ๐ญ> <๐ฉ๐ต๐ฎ๐ญ ๐ญ๐ข๐ฏ๐จ="๐ฆ๐ฏ"> ๐๐ฆ๐ญ๐ญ๐ฐ ๐ธ๐ฐ๐ณ๐ญ๐ฅ </๐ฉ๐ต๐ฎ๐ญ>`



### RENDERING <a href="#rendering" id="rendering"></a>

Now your browser should have gotten a response to its request, usually in the form of HTML and CSS. HTML and CSS are markup languages that your browser can interpret to load content and style the page. Rendering and laying out HTML/CSS is a very tricky process, and rendering engines have to be very flexible so that an unclosed tag, for example, doesnโt crash the page.

The request might also ask to load more resources, such as images, stylesheets, or JavaScript. This makes more requests, and JavaScript may also be used to dynamically alter the page and make requests to the backend. For example, the button on this page uses Javascript to show and hide the solution.

More and more, web applications these days simply load a bare page containing a JavaScript bundle, which, once executed, fetches content from APIs. The JavaScript application then manipulates the DOM to add the content it loaded.

![](../../.gitbook/assets/1644338439288.jpeg)
