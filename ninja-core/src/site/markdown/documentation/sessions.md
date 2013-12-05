Sessions
========

Intro
-----

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


Configure your session
----------------------

 You can configure your session via 
    /**
     * Prefix used for all Ninja cookies.
     * 
     * Make sure you set the prefix in your application.conf file.
     * 
     * */
    final String applicationCookiePrefix = "application.cookie.prefix";

    /**
     * Enables session/cookie sharing between subdomains. For example, to make cookies valid for
     * all domains ending with ‘.example.com’, e.g. foo.example.com and bar.example.com:
     */
    final String applicationCookieDomain = "application.cookie.domain";

    /** Used to verify client side cookie for instance. */
    final String applicationSecret = "application.secret";

    /**
     * Time until session expires.
     */
    final String sessionExpireTimeInSeconds = "application.session.expire_time_in_seconds";

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


