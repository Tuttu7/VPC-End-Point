### Creating an VPC endpoint to access your S3 bucket over private network without requirng Internet access.

### - Basic set up :

 * VPC  - 10.100.0.0/16
 * 2 subnets 
   *  Public Subnet A - 10.100.0.0/24
   *  Private Subnet B - 10.100.1.0/24
 
* 2 Ec2 instances created in each of these subnets
  * EC2-A Public
  * EC2-B Private

* Internet Gateway - igw 
* Route tables 
  * Route table for public subnet - public-route
  * Route table for private subnet - private-route
  
 * S3 bucket and an object (file) created inside it
   * Bucket name - privatebucket11
   * File name - testpage.html
 

### - My public subnet is connected to Internet via Internt gateway 

```
Route tabel entries :

Destination  Target                  Status  Propagated

10.100.0.0/16  local                 active   No

0.0.0.0/0    igw-0fe87224a14c13ef4   active   No
```
### - My private subnet only have the local route
```
Route table entries :

Destination     Target  Status Propagated

10.100.0.0/16   local   active  No
```
### The instances launced in the private subnet cannot connect to the internet. Now lets go to the Ec2 instance (Ec2-A) and from there connect the other Ec instance (Ec2-B).

### From the Ec2 instance B we can see that we are not able to download any content from an S3 bucket or not able to access internet.
```
[root@ip-10-100-1-231 ~]# wget https://privatebucket11.s3.us-east-2.amazonaws.com/testpage.html
--2019-11-02 17:32:10--  https://privatebucket11.s3.us-east-2.amazonaws.com/testpage.html
Resolving privatebucket11.s3.us-east-2.amazonaws.com (privatebucket11.s3.us-east-2.amazonaws.com)... 52.219.105.58
Connecting to privatebucket11.s3.us-east-2.amazonaws.com (privatebucket11.s3.us-east-2.amazonaws.com)|52.219.105.58|:443...

[root@ip-10-100-1-231 ~]# curl google.com
```

### - Creating an VPC End point
> VPC > End Points > Create End Point > Select the service name for which you need to create an end point gateway > S3 end point service > Select the route table (Route table assosiated with the private subnet)  > Create endpoint

### Once the endpoint has been created we can cross verify via going back to the route tables > select the route table assossitaed with the private subnet.

### You can see the following entries :
```
Destination             Target                  Status  Propagated
pl-7ba54012             vpce-07bde6b7f5d9f58b9  active  No
```

### Then go back to the Ec2-B instance and try to download the content from the S3 bucket. Make sure that the S3 bucket and the object (file) inside it has the public access enabled.
```
wget https://privatebucket11.s3.us-east-2.amazonaws.com/testpage.html
--2019-11-02 17:40:09--  https://privatebucket11.s3.us-east-2.amazonaws.com/testpage.html
Resolving privatebucket11.s3.us-east-2.amazonaws.com (privatebucket11.s3.us-east-2.amazonaws.com)... 52.219.104.152
Connecting to privatebucket11.s3.us-east-2.amazonaws.com (privatebucket11.s3.us-east-2.amazonaws.com)|52.219.104.152|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 10 [text/html]
Saving to: ‘testpage.html’

100%[============================================================================================>] 10          --.-K/s   in 0s      

2019-11-02 17:40:09 (291 KB/s) - ‘testpage.html’ saved [10/10]
[root@ip-10-100-1-231 ~]# wget google.com
--2019-11-02 17:40:55--  http://google.com/
Resolving google.com (google.com)... 172.217.8.174, 2607:f8b0:4009:80e::200e
Connecting to google.com (google.com)|172.217.8.174|:80...
[root@ip-10-100-1-231 ~]# curl google.com

```
### We are now able to download the content from the S3 bucket without connecting to the Internet 





