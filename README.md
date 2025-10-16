**Read this in other languages: [English](README-EN.md), [中文](README-ZH.md).**

## Update Log
Akamai has updated to version 3, and my API now supports version 3 of Akamai.
![image](https://github.com/user-attachments/assets/f3ec6f56-9bbd-4528-b082-7a3fc538925d)
Telegram: [xwggnp](https://t.me/xwggnp) 

### A new sbsd type has been added
![image](https://github.com/user-attachments/assets/27dd5e9c-d880-4b84-8dab-8ef9ae243b61)


## 1. Introduction
Akamai's anti-crawling system is a product designed to protect websites from attacks and ensure API data security. Currently, it has two methods of protection. One is protection at the HTTPS request level, and the other is to collect current browser information for risk control interception. Different sites and interfaces have varying degrees of risk control interception..

## 2. Bypassing the ja3 fingerprint at the HTTPS request level
There are many articles online about bypassing ja3. I recommend a third-party Python library for bypassing ja3, [curl_cffi](https://github.com/yifeikong/curl_cffi).

## 3. `sensor_data` risk control verification process
The site analyzed this time is [maersk.com](https://www.maersk.com/). The main verification is done using the **_abck** cookie value. This is returned when submitting the **sensor_data** parameter.
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34896050/1685808309127-afadf7a0-2646-4760-8243-6dcb9981a205.png#averageHue=%23777d85&clientId=ud24bafdf-8053-4&from=paste&height=528&id=u7a468abe&originHeight=1056&originWidth=2280&originalType=binary&ratio=2&rotation=0&showTitle=false&size=772599&status=done&style=none&taskId=u18ace2ff-0f00-4f4f-9341-e06ff92d2f1&title=&width=1140)

If the **sensor_data** value is valid, a valid **_abck** will be returned. Then, you can happily request the content of the corresponding website. However, **_abck** has a time limit. After being intercepted by a 403 error, you need to request it again.

## 4. Analysis of the `sensor_data` Parameter

Let's now analyze how the `sensor_data` parameter is generated.

![Image omitted for clarity](https://cdn.nlark.com/yuque/0/2023/png/34896050/1685808732764-3bfa0b39-494c-4465-ad1f-fa60d81ab428.png)

The JavaScript code responsible for generating this parameter is shown in the image above. It's obfuscated, but no worries, we can employ **AST** (Abstract Syntax Tree) to de-obfuscate it. For those interested in the nuances of AST de-obfuscation, you may refer to an expert named Cai's AST knowledge base. Without going into the de-obfuscation process, I dived directly into debugging the code.

![Image omitted for clarity](https://cdn.nlark.com/yuque/0/2023/png/34896050/1685808967444-dff2a27e-b378-44b8-b956-ec6a54f2518b.png)

The `sensor_data` parameter is formed by concatenating and encrypting a 58-element array, as shown above. Let's delve deeper to understand what each element in this array represents.

![Image omitted for clarity](https://cdn.nlark.com/yuque/0/2023/png/34896050/1685809243314-a6ab8099-112f-4ccb-a729-f912bcc393fd.png)

These images showcase the array's detailed content. 

![Image omitted for clarity](https://cdn.nlark.com/yuque/0/2023/png/34896050/1685809090261-e3dff937-bc1c-4935-a77f-9dfded1fda03.png)

By debugging the JS code, I was able to discern the meaning and significance of several parameters within the array. Crucially, I discovered that the **canvas** fingerprint and **motion trajectory** were the most vital components. Moreover, it's essential to note that the **canvas fingerprint can't be randomly forged**. To bypass this, one might collect real browser canvas fingerprints to use as replacements. As for motion trajectories, algorithms can be designed to simulate genuine mouse movements, clicks, and other behaviors.

:::info
###### When dealing with high concurrent requests, it's imperative that simulated browser fingerprint data and motion trajectory data appear genuine. Also, it's necessary to switch IPs dynamically. Making numerous requests from a single IP will lead to its blacklisting. Another crucial insight is that if the generated parameters aren't realistic enough, the Akamai system might allow them but will delay the request processing time. This latency becomes evident in some critical interfaces.
:::

## 5. Generation of the `akamai-bm-telemetry` Parameter

This parameter is included in the request headers. Its generation is relatively straightforward as it's derived from the `sensor_data` parameter, and is encoded using base64. Let's examine it.

![Image omitted for clarity](https://cdn.nlark.com/yuque/0/2023/png/34896050/1685810544615-ef1ab010-6d7b-4375-ad49-509d6c38acff.png)

The translation of the relevant code reveals:

![Image omitted for clarity](https://cdn.nlark.com/yuque/0/2023/png/34896050/1685810585928-04c403de-2bc8-496f-a9b5-064bfa8db473.png)

This is how it's generated.

## 6. Results Showcase
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34896050/1685810646236-8a658740-246d-4f1d-98de-85cbf3f66e95.png#averageHue=%234b4949&clientId=ud24bafdf-8053-4&from=paste&height=934&id=u7dc07597&originHeight=1868&originWidth=1806&originalType=binary&ratio=2&rotation=0&showTitle=false&size=1297220&status=done&style=none&taskId=u2aa30fa7-2e49-48e3-a3d3-00c27a36c85&title=&width=903)

With a concurrency of 100, the pass rate is 100%. As you can see, if the **sensor_data** is generated more realistically, the pass rate is very high, bypassing Akamai's system in less than a second. If the confidence is not very high, it will slow down, and it might take up to 10 seconds to pass. Of course, this also depends on the IP address used.

## 7. Business Cooperation
I've already encapsulated the algorithm for generating **sensor_data** into an API, which can be called. If you don't want to research it yourself, you can contact me for API call services. Of course, there's a fee.
My contact details:


Telegram: [xwggnp](https://t.me/xwggnp) 


