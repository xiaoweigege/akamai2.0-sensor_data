
**其他语言版本: [English](README-EN.md), [中文](README-ZH.md).**


## 1. Introduction
Akamai's anti-crawling system is a product designed to protect websites from attacks and ensure API data security. Currently, it has two methods of protection. One is protection at the HTTPS request level, and the other is to collect current browser information for risk control interception. Different sites and interfaces have varying degrees of risk control interception.

## 2. Bypassing the ja3 fingerprint at the HTTPS request level
There are many articles online about bypassing ja3. I recommend a third-party Python library for bypassing ja3, [curl_cffi](https://github.com/yifeikong/curl_cffi).

## 3. `sensor_data` risk control verification process
The site analyzed this time is [maersk.com](https://www.maersk.com/). The main verification is done using the **_abck** cookie value. This is returned when submitting the **sensor_data** parameter.
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34896050/1685808309127-afadf7a0-2646-4760-8243-6dcb9981a205.png#averageHue=%23777d85&clientId=ud24bafdf-8053-4&from=paste&height=528&id=u7a468abe&originHeight=1056&originWidth=2280&originalType=binary&ratio=2&rotation=0&showTitle=false&size=772599&status=done&style=none&taskId=u18ace2ff-0f00-4f4f-9341-e06ff92d2f1&title=&width=1140)

If the **sensor_data** value is valid, a valid **_abck** will be returned. Then, you can happily request the content of the corresponding website. However, **_abck** has a time limit. After being intercepted by a 403 error, you need to request it again.

## 4. Analysis of the `sensor_data` parameter
Now let's analyze how the **sensor_data** parameter is generated.
[...Images preserved...]

:::info
###### For high concurrency, the simulated browser fingerprint information and motion trajectory data must be sufficiently realistic. Additionally, IP addresses need to be dynamically changed. If one IP address makes requests at a high concurrency rate, it will be blocked. Another key piece of information is that if the generated parameters aren't realistic enough, the Akamai system will allow them through but will slow down the request time. This can be evident in some crucial interfaces.
:::

## 5. Generation of the `akamai-bm-telemetry` parameter
This parameter is carried in the headers of the interface. It's relatively simple to generate, based on the **sensor_data** parameter and encoded with base64. Let's take a look:
[...Images preserved...]

## 6. Results Showcase
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34896050/1685810646236-8a658740-246d-4f1d-98de-85cbf3f66e95.png#averageHue=%234b4949&clientId=ud24bafdf-8053-4&from=paste&height=934&id=u7dc07597&originHeight=1868&originWidth=1806&originalType=binary&ratio=2&rotation=0&showTitle=false&size=1297220&status=done&style=none&taskId=u2aa30fa7-2e49-48e3-a3d3-00c27a36c85&title=&width=903)

With a concurrency of 100, the pass rate is 100%. As you can see, if the **sensor_data** is generated more realistically, the pass rate is very high, bypassing Akamai's system in less than a second. If the confidence is not very high, it will slow down, and it might take up to 10 seconds to pass. Of course, this also depends on the IP address used.

## 7. Business Cooperation
I've already encapsulated the algorithm for generating **sensor_data** into an API, which can be called. If you don't want to research it yourself, you can contact me for API call services. Of course, there's a fee.
My contact details:


Telegram: [xwggnp](https://t.me/xwggnp) 

