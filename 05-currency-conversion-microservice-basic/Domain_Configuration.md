### How To configure Domain

- Go to https://hpanel.hostinger.com and create a domain 
  in my case i created agromanipalfood.shop domain 
  
- Go to Google cloud and search Cloud DNS
  create a Zone with name agro-manipal-food-shop with below configuration:
    Zone Name: agro-manipal-food-shop
	DNS name : agromanipalfood.shop.
	Description : Shopping DNS
	Type : Public
	DNSSEC : Off
	Cloud Logging : Off
	
- Inside Zone create record set with bellow configuration:
    -------------------Record Set 1----------------------
    DNS name: www (.agromanipalfood.shop. by default will be there)
    Resource record type : A
    TTL : 5
    TTL unit : minutes
    IPv4 Address : <EXTERNAL IP OF INGRESS>
    ---------------------Record Set 2----------------------
    DNS name: www (.agromanipalfood.shop. by default will be there)
    Resource record type : A
    TTL : 5
    TTL unit : minutes
    IPv4 Address : <EXTERNAL IP OF INGRESS>
    
 - After Above step is complete now associate the name-servers
   inside Zone of agro-manipal-food-shop you will see Registrar Setup click it and copy paste the text to name-server Hostinger
   Ex:
   DNS/Nameservers
	ns-cloud-b1.googledomains.com
	ns-cloud-b2.googledomains.com
	ns-cloud-b3.googledomains.com
	ns-cloud-b4.googledomains.com
	
- After above step wait for max 24H and you can hit the URL : http://www.agromanipalfood.shop/currency-exchange/from/EUR/to/INR
  OR 
  http://agromanipalfood.shop/currency-exchange/from/EUR/to/INR