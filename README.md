https://github.com/thegitaccount/course-project-1

The web application has a login page and after logging in you can upload, download or delete your own files. There's also a script for demonstrating of a brute force attack. The applications has some security flaws which will be explained later in this essay. The description of each individual flaw is not meant to be comprehensive but giving the basic idea behind each flaw. Detailed descriptions can be found on owasp.org. There are no extra installations needed for testing the flaws if you have completed securing software course (you need django and python3).



Flaw 1: Broken Access Control (OWASP A01:2021)

https://github.com/thegitaccount/course-project-1/blob/2a0668d207690ff31c89a96c0961f5bea068ed3a/server/pages/views.py#L7-L38

Description:

Access control ensures that a user cannot see, modify or delete other users data without permission. The principle is to deny access by default and then only grant access to particular resources that a user needs (and nothing else). It should not be possible to bypass access control by modifying the URL.

How to fix:

Example code has broken access control because it let's users to download other users files. This can be tested by logging in as admin (password is sunshine) and trying url http://127.0.0.1:8000/download/24. It will download passwords.txt file from someone else's files.

This could be avoided by modifying the code in views.py so that it'll check if the user requesting the file is the owner of the file. Replacing lines 7 -38 with the lines 39 - 80 would fix this issue.



Flaw 2: Cryptographic Failures (OWASP A02:2021)

https://github.com/thegitaccount/course-project-1/blob/fd628e7f1bf6216d26827bdde02238b510acca19/server/config/settings.py#L23

Description:

This kind of flaw is related to lack of cryptography and leads to exposure of sensitive data. Applications should never transmit data in clear text meaning protocols like http and ftp should not be used. 

How to fix:

Instead of hardcoding the secret key in settings.py we should load it from an environment variable or from a file. An example of the fix can be seen in lines 25 - 29.



Flaw 3: Identification and Authentication Failures (OWASP A07:2021)

https://github.com/thegitaccount/course-project-1/blob/fd628e7f1bf6216d26827bdde02238b510acca19/server/config/settings.py#L103-L116

Description:

Identification and authentication failures happen because of authentication weaknesses. This flaw may lead to success e.g. in automated brute-force attacks where a hacker uses well-known usernames and passwords. The developer of this web application has forgotten to uncomment the lines 103 -116 in settings.py and therefore all passwords are weak.

You can try a brute force attack now. First run the server and then the script in src directory:

python3 hackpassword.py http://127.0.0.1:8000 candidates.txt 

This will end up in finding the right password for admin.

How to fix:

First the authentication password validators should be uncommented, lines 103 - 116 in settings.py. 

After that the default admin page url should be changed to something else. Below in urls.py is an example. One should comment out line 21 and uncomment line 22.

https://github.com/thegitaccount/course-project-1/blob/2a0668d207690ff31c89a96c0961f5bea068ed3a/server/config/urls.py#L21-L22

If you changed the url the automated script isn´t working anymore because the admin page has moved. You should also force users to change their weak passwords to strong ones.

You could also install capthca (completely automated public turing test to tell computers and humans apart) or Google's recaptcha. Another good way to fix this flaw would be to install django honeypot which creates a copy of an admin page and will also log all unauthrorized login attempts. 



Flaw 4: Security Logging and Monitoring Failures (OWASP A09:2021)

https://github.com/thegitaccount/course-project-1/blob/fd628e7f1bf6216d26827bdde02238b510acca19/server/config/settings.py#L139-L163

Description:

One common flaw in web applications is caused by insufficient logging and monitoring. OWASP recommends all web developers to ensure that all login, access control and server-side input validation failures will be logged in order to identify suspicious activity. Without proper logging malicious activity can go on undetected for a very long time. This gives attackers a lot of time to cause damage.

The configuration on lines 139 - 163 in settins.py will create a security.log file with login attempts in it. This log file isn't good enough for logging and monitoring purposes. It's hard to tell which login attempt is legitimate and who's behind it. Suspicious login attempts might get unnoticed as the file has also lines that have nothing to do with login attempts.

How to fix:

For example django honeypot, which creates a copy of admin login page will log all unauthorized login attempts. It pulls the users ip address from the REMOTE_ADDR request header. You can also configure the honeypot to send email if there's suspicious login activity. 



Flaw 5: Security Misconfiguration (OWASP A05:2021)

https://github.com/thegitaccount/course-project-1/blob/fd628e7f1bf6216d26827bdde02238b510acca19/server/config/settings.py#L33

Description:

This kind of flaw typically takes place if an application has default accounts and passwords enabled. Another example is out-dated vulnerable software. One cause for this kind of flaw is error handling which reveals too much information about the system.

How to fix:

In the web application example the default setting for debugging purposes is set to True. If you changed the default admin login page earlier (flaw 3) the server will now tell the new url in an error message. You can for example type:

http://127.0.0.1:8000/some_random_string

and the server will tell the new admin page path, among other paths.

To fix this replace the line 33 with the line 38 and the server will tell:

Not Found
The requested resource was not found on this server.
