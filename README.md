Backend cannot connect to MySQL is why your container exits.

This happens because your containers are not on the same Docker network.
Why this happens

By default:

Each docker run uses the default bridge

Containers cannot resolve each other by name

DNS works only on user-defined networks

So your backend tries:
