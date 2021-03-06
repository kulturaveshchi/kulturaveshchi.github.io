---
layout: post
title: Modifying the design
---

Inevitably, some modifications in the design proved necessary in the course of 
implementation.

## Keeping secrets ##

When I began writing the code to authenticate with Twitter (using the [OmniAuth
gem](https://github.com/omniauth/omniauth)), I realised that I would need some
way to keep the API secrets out of band. What I'm doing is storing them in the
environment, which is loaded from the file /etc/environment, which is in turn
generated by an Ansible task from a hash in group_vars (and I've added 
group_vars to my gitignore so that the credentials don't get added to version
control). 


## Hashing the authentication token ##

In my initial design, I used the Twitter oauth token as an authentication token
for my own clients. However, this meant that the authentication token was stored in
plain text in the database, which is a security problem (specifically, if
an attacker gets read-only access to a database containing useable tokens, they
can then get write access to the app, which is a privilege escalation). Note that
it isn't a problem to store the Twitter oauth token in plain text in the database
if I'm only using it to contact Twitter - the token is only useable in conjunction
with my application secret, which, as explained above, is not stored in the database.

So, rather than re-using the Twitter token, I'm now generating a token myself, 
using the ruby SecureRandom facility. I've chosen to use a token that is 128 
bits long, which (by the authority of random googling) is apparently 
sufficiently long that the hash cannot practically be brute forced. For that 
reason, I'm using a single round of SHA256 as the hash function, rather than 
bcrypt, as my understanding is that bcrypt is only necessary to increase the 
difficulty of brute-forcing comparatively low-entropy tokens (such as 
passwords). 



