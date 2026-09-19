# Cookies

When You Login to some server, Some data has to be stored about user  
Like his preferences. Theme, fontsize or Login information etc.  
This makes sense to store this data at users end insted of storing this  
at server.  

as Servers are stateless in this case,they cannot remember and HTTP  
request is also stateless and sending all the data manually everytime  
is not possible  

Cookies come into picture so that browser automatically can send cookie  
(data) when ever request is made again to same origin  
Solves one but introduces another problem Where CSRF is used.  


# CSRF (Cross-Site Request Forgery)  

There is a scenario where other origins can cause a request to be made  
and the request is considered as if you made it because cookies are  
attached by the browser automatically before sending the request.  

To avoid this we save a CSRF token at the client and the client attaches  
it manually (similar to JWT).  

If the token matches, the request is accepted, avoiding the vulnerable state.  


# CORS (Cross Origin Resourse Sharing)

Backend is already capable of Authenticating, Validating and weather to  
send or not send any response. This CORS is simple browser policy even  
when backend sends the data it checks for header  

`Access-control-allow-origin` setted by cors at backend and value is  
matched with current domain and browser can reject the origin's  
Javascript to access the response if the header origin is not matched  
or header is missing. 

Just a browser Policy and nothing to do with security
