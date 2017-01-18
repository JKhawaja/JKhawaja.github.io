## Hash Collisions in Dropbox Passwords? No Problem.

Recently, Dropbox posted their [new process for securing user's passwords](https://blogs.dropbox.com/tech/2016/09/how-dropbox-securely-stores-your-passwords/). A critical reader might realize that SHA512-ing all passwords of arbitrary length could potentially lead to hash collisions. Well, they would be correct. But, why would that be a security concern? Hash collisions basically equate to: every password will now have multiple qualifying text strings. In other words: an attacker has an increased number of possible text strings that they can guess that the system will accept as equivalent to a user's password (after they are hashed).

Yikes! That sounds terrible! In fact, some simple mathematics shows us: if we allow passwords up to 256 bytes in length over the same alphabet of characters that the SHA512 algorithm will hash your text too, then every user's password could have as many as 191 other pieces of text (besides their actual password) that the system will accept as being their password! So, how has Dropbox not been pwned yet?

Thankfully, the mathematics does not stop there. We also have to consider that while an attacker may have an increased penetration window, they also have a massively decreased probability of getting through that window. For example, if we decided to try to avoid hash collisions and only allow our user's passwords to be of a length no greater than the length of SHA512 hashes, then our attacker would only have 1 shot in a |Alphabet^64| chance of guessing our password text. But, here is the cool part ... That number is WAY larger than if we allow hash collisions and give our attacker a 192 in |Alphabet^256| chance of guessing a valid password text.

$$\frac{1}{|A^{64}|} >> \frac{192}{|A^{256}|}$$

So, even with allowing hash collisions we are still getting increased security benefits (in terms of probabilities). Pretty neat, huh?

Btw, my Mathematics is oversimplified and very possibly incorrect! However, the basic conclusion still remains true. You will have to forgive me as this post was thought up in an hour and written in a quarter of that time. Thanks for reading!