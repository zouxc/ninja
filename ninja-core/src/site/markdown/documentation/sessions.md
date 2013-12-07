Sessions
========

# Intro

A session is a package of information that always maps to one particular user of your application. 
It is persistent over many connects. For instance - A user logs into your application today 
and comes back tomorrow. Using sessions you will be able to recognize the connects as belonging to the
same user.

Session handling can be achieved in many ways. Php and many Java stacks use a cookie with a "secret" key
that then maps to a session that is stored on a db or filesystem. These approaches are called server side sessions.

Ninja does not do it that way. Instead Ninja uses so called client-side sessions. The cookie itself stores
the information you want to attach to that session. The cool thing is: Scaling Ninja servers becomes
much simpler. You do not need sticky sessions or any filesystem that has to be fast enough to
deliver sessions to your servers. The bad thing is that a Ninja session can only contain very little
information (up to around 4k).

By the way - Ninja did not invent the concept of client side sessions - The praise goes to Rails (afaic). 

More on sessions: (http://en.wikipedia.org/wiki/Session_(computer_science))

# Dealing with sessions

A user sessions is valid for a certain amount of time. Once a user comes back Ninja automatically prolongs Ninja's session.

Usually you modify your sessions in your controller methods. You can obtain a sessions via the Context object that
has a method getSession(). But you can also directly inject the Session into your controller.

<pre class="prettyprint">

	// Direct Session injection of the current session
    public void controllerMethod(Session session) {
       
        session.put("username", "kevin");

    }
    
    // Context also offers access to the current session of this request
    public void controllerMethod(Context context) {
       
        context.getSessionCookie().put("username", "kevin");

    }

</pre>

Session offers methods to put, delete and get data. Please refer to JavaDoc for more detailed
information.

Session data is also available as built-in variable in the html templating system.
For instance ${session.username} allows you to access the username of the session inside your
html template. 


# Configure your session

You can configure Ninja's session handling in your application.conf file.

The following parameters are available.

### application.cookie.prefix

Allows you to set the prefix of the session cookie. By default this is NINJA_ (and your cookies
will be called NINJA_SESSION).

### application.cookie.domain

Enables session/cookie sharing between subdomains. For example, to make cookies valid for
all domains ending with ‘.example.com’, e.g. foo.example.com and bar.example.com.

### application.secret

This is the secret key used to sign the cookie. You really have to make sure that the key is kept secret.
To generate a new one simply delete the key and start Ninja on the command line. This will generate
a new random key.

### application.session.expire_time_in_seconds

This defines how long a session will be valid. 
If a user comes back the session validity is automatically prolonged.

If you want to invalidate a session you need to check that via a persistent storage on the server side.
Usually you want to sue a database and/or memcache to check if a user login is valid or not under
certain circumstances.


### application.session.send_only_if_changed

Send session cookie only back when content has changed.



    /**
     * Send session cookie only back when content has changed.
     */
    final String sessionSendOnlyIfChanged = "application.session.send_only_if_changed";


    /**
     * Used to set the Secure flag if the cookie. Means Session will only be
     * transferrd over Https.
     */
    final String sessionTransferredOverHttpsOnly = "application.session.transferred_over_https_only";


    /**
     * Used to set the HttpOnly flag at the session cookie. On a supported
     * browser, an HttpOnly session cookie will be used only when transmitting
     * HTTP (or HTTPS) requests, thus restricting access from other, non-HTTP
     * APIs (such as JavaScript). This restriction mitigates but does not
     * eliminate the threat of session cookie theft via cross-site scripting
     * (XSS).
     */
    final String sessionHttpOnly = "application.session.http_only";

Session security
----------------

A Ninja session is a hash of key/values, signed but not encrypted. 
That means that as long as your secret is safe, it is not possible for a third-party to forge sessions.

The secret is stored as key <code>application.secret</code> at <code>conf/application.conf.</code>

That way - several servers sharing the same secret can handle any request coming from your users. 
That's the reason why scaling is simple.

<strong>It is very very important to keep the application.secret private.</strong>

Do not commit it in a public repository, and when you install an application written by 
someone else change the secret key to your own. 

When deploying it is also really useful to use an external configuration containing production settings -
and a special application.secret that is not used in regular development. You can point to an alternate
configuration by using a system variable:

<code> ... -Dninja.external.configuration=/mydir/deployment.conf"</code>.


Generating a new secret
-----------------------

You can generate a random new secret in development mode by simply deleting application.secret from
your conf/application.conf file. When you restart your server Ninja will generate a new secret and 
add it to conf/application.conf.


What data to store in session
-----------------------------

Ninja sessions currently are not encrypted. Therefore you should not store any 
critical information. Storing a user id, oder username is fine. Storing
credit card information is really bad practise.


