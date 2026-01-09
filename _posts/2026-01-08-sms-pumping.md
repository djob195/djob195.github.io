---
layout: post
title: "SMS pumping Case"
date: 2026-02-17 09:00:00 -0600
categories: cases
tags: security ddos auth
---
Nowadays, many authentication systems aim to provide a simple user experience, typically based on requesting a **one-time password (OTP)** through a channel previously associated with the client, such as SMS, WhatsApp, or other messaging platforms.

However, this approach has become a significant security vulnerability for many companies due to the emergence of an attack known as **SMS Pumping**. This type of attack involves sending massive numbers of SMS requests, which can dramatically increase a company’s billing. In some cases, attackers may even gain indirect financial benefits through agreements with telecom operators or messaging providers, or they may impact the operational availability of the targeted businesses.

![sms pumping](/assets/posts/2026-01-08-sms-pumping/sms-pumping.png "sms pumping")

Below, I share a real case I worked on, where this type of attack occurred, along with the solution implemented to effectively mitigate it.

## Case

A company had integrated an OTP-based authentication service, which relied on a third party for sending access codes via SMS or WhatsApp. To access the environment, the system sent an unencrypted request that included the phone number and country code.

Due to this weakness, a fraudster was able to intercept the request and carry out a massive OTP request attack targeting multiple random numbers from a country whose telecom infrastructure did not allow verification of whether the numbers were valid. As a result, the company’s billing service increased significantly.


![flow company](/assets/posts/2026-01-08-sms-pumping/diagram2.png "flow company")

## How was the issue discovered?

Continuous monitoring of the SMS billing service revealed that, in the preceding days, the billed volume had significantly exceeded the normal daily operational average. Additionally, the provider reported that the charges corresponded to OTP services sent to a specific country and exhibited an atypical message-sending pattern: large numbers of SMS were being sent per minute, outside normal operating hours, which did not correspond to human behavior.

This situation triggered security alerts and led to intensified monitoring of the company’s authentication service through the firewall. During this review, it was confirmed that OTP service requests were being repeatedly called from multiple IP addresses worldwide.

## What were the temporary solutions?
The company was racing against time, as SMS billing was increasing every minute, and at the same time, they could not fully block the authentication service without affecting normal operations. In response, temporary solutions were implemented to buy the team time and develop a permanent solution:

1. **Blocking non-operational countries:**
A blacklist was created in the firewall to block requests originating from countries where the service was not operational. This reduced the volume of requests, but it was not completely effective, as the attacker rotated IPs from countries with active operations, which could not be blocked.

2. **Blocking based on request headers:**
By analyzing the headers of each request in the firewall, it was observed that fraudulent requests shared common characteristics, such as device language, operating system type, and others. A blocking rule was implemented for IPs matching these headers, which prevented excessive billing for several days. However, the attacker adapted their headers to mimic legitimate clients, bypassing this temporary measure.

3. **Disabling the OTP-by-SMS service for the affected country:**
As a final measure, the OTP-by-SMS service was disabled for the affected country, and customers were notified about alternative authentication methods, such as WhatsApp or phone calls. Although these alternatives were more expensive and less frequently used, they were not affected by the attack, as they validated the phone numbers and did not incur billing if the number was invalid.

## Solution

To mitigate SMS pumping attacks, a common recommendation is to implement IP-based throttling, limiting the number of requests per minute. However, this measure could not be applied effectively, as the attack also behaved like a DDoS, making this approach inefficient.

Another alternative considered was to implement pre-validation before sending the OTP, such as a CAPTCHA. However, this solution was not feasible, as it would negatively impact the user experience and required prior approval. Therefore, both ideas were adapted to create a similar solution that would mitigate the attack without affecting the user experience.

### Implementation Steps

**Step 1:**
The CAPTCHA concept was adapted to the existing authentication flow. Before requesting the OTP, the client had to make an additional request to obtain a unique UID, which functioned as a temporary password. This UID validated the OTP request and had an expiration time of 1 minute, acting similarly to a requests-per-minute control. Finally, the UID was included in the body of the OTP request.

**Step 2:**
Symmetric AES encryption was applied to the request body. The secret key and other encryption parameters were configured in Firebase Remote Config, allowing them to be used both on the frontend (mobile applications) and the backend (authentication service).

This approach offered the advantage of quickly and easily rotating the keys without needing to wait for approval of new versions on the App Store or Play Store, ensuring the protection of sensitive data.

## Results

The solution fully resolved the problem. After its implementation, daily SMS billing returned to normal operational levels. Additionally, the attacker faced significant difficulties in replicating requests because:

* They would need to understand the new authentication flow required to request an OTP.
* They would have to identify the type of encryption used to protect the data.
* They would need to comprehend the new structure of the request body.
* Even if they managed to obtain the secret keys and AES encryption parameters, their usefulness would be limited, as these keys were rotated frequently and only valid for short periods.
* Furthermore, OTP requests were unique and single-use; therefore, even if the attacker obtained a valid request, it could not be reused.

It is worth noting that, to further strengthen the OTP process, all firewall blocking rules implemented during the temporary solutions were maintained. Additionally, the OTP-by-SMS service for the affected country was reactivated, providing a higher level of security without compromising operations.