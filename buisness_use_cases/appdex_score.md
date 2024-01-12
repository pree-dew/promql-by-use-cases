**Use Case**

How to find Appdex score of my HTTP application?

**Explanation**

Appdex stands for Application Performance Index. It's a measure of application performance with respect
to user satisfaction. 

**Why Appdex score is a better measure**

Definition of application performance varies as per different personas. Foe engineer it's response time, 
for SRE it's uptime, for product it's user retention on product. These definitions are varying as per the 
roles or priorities of these roles. In such cases if we have to find a uniform way to measure application
performance across multiple services, applications, teams then how to do that.
**Appdex** solves that problem by quantifying **user experience** to a number, as at the end for any buisness
all roles are trying to find user experience by using different measurements.

To define appdex in terms of user experiences, we take three different category of users.
1. **Satisfied** - One who enjoys the application experience without any hicups or slowness.
2. **Tolerating** - One who face some lag or slowness but still keep using the application without complaining
3. **Frustrated** - One who is abandones the application due to lag or slowness.

Appdex measures satisfied, tolerating and frustated users on the basis of application response time.

**How to find Appdex score for my application**

- Define a satisified user response time threshold, for example, a happy customer whould get response with
        in 1 sec.
        
- 4 times the satisified user threshold defines the threshold for tolerating users. So by using above example
        anything time from 1 sec to 4 sec defines the tolerating user response time. Anything above that defines
        frustated user response time.
        
- Find appdex user by:

  ```        
  Appdex score = (
                    No. of satisfied user +
                    (0.5) * No. of tolerating user
                     + 0 * No. of frustated user
                ) / Total users
  ``` 

**Example for Appdex maths**

For any application, no of users can be quantifies by no. of requests. 

Assumptions:
- Application received a throughput of 1000 requests.
- Satisfied user thrreshold is 1 sec
- No. of requests finished within 1 sec = 600
- No. of requests finished within 1 to 4 sec = 200
- No. of request finished greater than 4 sec = 200

Appdex Score will be:

- Satisified users : (600 * 1 = 600)
- Tolerating users : (200 * 0.5 = 100)
- Frustating users : (200 * 0 = 0)
  
  ```
  score = (600 + 100 + 0 ) / 1000 = 0.7
  ```
