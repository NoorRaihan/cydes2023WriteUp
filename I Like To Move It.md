Cydes 2023 CTF Competition

This challenges consists multiple vulnerabilities that includes SQL Injection and SSTI.

When I opened the challenge, they gave us a link to the web and a zip that contains the source code of the challenge which is means it will be a whitebox challenge.

![[Pasted image 20230711222740.png]]

When I opened the app.py, we can see that the web is running using flask library.

![[Pasted image 20230711222845.png]]

Let's try to install all the requirements and try to run the app locally if we can.

![[Pasted image 20230711223037.png]]

Wow!! We can run the app locally which means we can debug the app directly from our local.

As we can see, we have the login page but we did not know the login credentials for the login page. Forgot password also not be configured to hit any file. Therefore let's take a look at the code.

This is where it will trigger the "/" route.
![[Pasted image 20230711223417.png]]
Nothing interesting... Let's check the login section.

![[Pasted image 20230711223451.png]]

Interesting... We can see if the remote address is localhost and the user is internal then it will create a new session and insert into the DB. Hurmmm it seems like we can't inject sql in this area.

Let's check init section to check the logic when the app is triggered.
![[Pasted image 20230711223707.png]]

It's looks like it will init the database and create a table. Maybe we can use this information later but we can see there is a guestinfo table. Let's check which part of the copde that will use that table.

![[Pasted image 20230711223901.png]]

NICE!! We got some developer's comment there then it means it got something cool in here xD. The code will check the request if x-Mode is in the request headers and value if 1 and 0 it will set g_access 1 and g_access2 to true.

If both *TRUE* then it will insert the data into guestinfo tables then update the table again to insert the email.

BINGO! The devs use the variable directly inthe sql query then it will be SQL INJECTION!! Let's try trigger the url. You can use anything you wanted to trigger but I use thunderclient ;)

![[Pasted image 20230711224357.png]]

Yeayyy!! We can hit the URL in POST mode. Now for the debug part let's try to check what's happening in the code.

![[Pasted image 20230711224601.png]]

Let's put print in line 70, 79 and 90 to check what's happening by send all the required data.

![[Pasted image 20230711224856.png]]
![[Pasted image 20230711224907.png]]

![[Pasted image 20230711224924.png]]

Why it did'nt hit BINGO? It's because the header turns out to be X-Mode instead of x-Mode. Therefore, I realised they use "in" method that will check if particular word is exists in the string. Let's try put something like bujangx-Mode1 and bunjangx-Mode2.

![[Pasted image 20230711225211.png]]

BINGO! We bypassed the x-Mode part. Now for the sql injection in name input then I can simply do insertion to the session table since the /home only check for the session.

payload: bujang' ; INSERT INTO activesessions VALUES('bujang', 'bujang') --

![[Pasted image 20230711225733.png]]

BINGOO BINGO!! Now let's change our cookies using cookie editor and we should able to access /home.

![[Pasted image 20230711225832.png]]

![[Pasted image 20230711225902.png]]

![[Pasted image 20230711225926.png]]


Tadaaaa! Now search for a file??
![[Pasted image 20230711230000.png]]
It's just simply printed out the search query. We need to check the code again...... Hurmmm

![[Pasted image 20230711230056.png]]
It will simply render our search query using render_template_string(). After few google searches...

![[Pasted image 20230711230207.png]]

Woohoo SSTI!! Here is the link to read https://medium.com/@nyomanpradipta120/ssti-in-flask-jinja2-20b068fdaeee

Let's try to print out some subclasses in the app environment.

![[Pasted image 20230711230438.png]]

Yeayyy! Let's find a class like subprocess.Popen to run command so we can escalte the vulnerabilities into RCE. 

The subprocess.Popen is in array index 421 in my local, let's create a payload to read the flag in the root path.

Payload: `{{''.__class__.__mro__[1].__subclasses__()[421]('ls',shell=True,stdout=-1).communicate()}}`

![[Pasted image 20230711231552.png]]
Flag captured!!









