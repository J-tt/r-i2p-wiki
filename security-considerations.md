# I2P Security Considerations
 
I2P is a great tool for online privacy & security, but like any software, it is not perfect. There are issues one needs to consider before and during use of I2P.
 
This is in no way a comprehensive guide to I2P security, and there may be some mistakes.
 
## Traffic Correlation
 
Traffic correlation is a problem with all anonymity networks, especially in low-latency networks like I2P.
 
Assuming one is not using a VPN, a user's Internet provider will know that they are using I2P, how much traffic they are relaying, and even potentially who/what they are communicating with if they can combine this information with other nodes in the I2P network, or with other ISPs data.
 
I2P-Bote, a sub-network inside I2P for anonymous mail exchange protects against this to some extent by obscuring the recipent of a message and by being high-latency.
 
### VPNs
 
VPNs can be used before connecting to the I2P network, but all this will do is obscure the fact that a user is using I2P from their ISP. If one lives in a country with decent human rights, this is not generally necessary, and [in some cases can even harm one's privacy](https://matt.traudt.xyz/posts/vpn-tor-not-mRikAa4h.html) (Related to Tor, but applies mostly the same to I2P)
 
## Browser Fingerprinting 

Eepsites are a popular feature of I2P, however these websites posses the ability to potentially fingerprint and track a user's web browser. Having JavaScript enabled is especially risky, however [CSS alone can be used to track someone too](https://matt.traudt.xyz/posts/how-css-alone-YF4ciVY6.html) (Related to Tor, but applies the same to I2P).
 
If a user's browser is sufficiently unique enough, this will greatly increase the ability for an adversary to track them, especially if they use the same/similar configuration on the clearnet.
 
It is recommended to configure the Tor Browser Bundle for I2P and to not resize the browser window.

## User Profiling
 
Besides browsers & traffic, a user can also be profiled based on their behavior. For example, if one uses a particular style of writing, posts at certain times of the day, reuses usernames & passwords, contact information, personal information, etc, this can be used to track a user cross-services, and even potentially to a clearnet identity.
 
For example, [language patterns were used to track down a pedophile in 2016](https://www.deepdotweb.com/2016/07/20/police-infiltrated-darknet-forum-hunt-pedophiles/).
 
One should be careful with the language they use and information they reveal.
 
## Malware
 
If a user's device is already compromised with malware, no software including I2P can effectively defend against it. This includes any malicious design in a user's OS or hardware.
 
Users should consider booting into a Live OS on trusted hardware to use I2P, if preexisting malware is a concern.
 
## Quantum Security & Future Exploits
 
If one's communications must stay secret forever, then they should worry about future exploits of cryptography, particularly with quantum computers. Most public key cryptography is vulnerable to quantum computers, however this is likely out of most user's threat model currently, and I2P has forward secrecy which would require an adversary to intercept & store **complete** streams until a suitable computer exists, which would be very difficult to do.

 
## Legal Considerations
 
As stated earlier, internet providers can see that a user is using I2P, which may be problematic if a user is in a location where encryption or online anonymity would draw suspicion. Remember, no encryption can protect against [a $5 wrench](https://xkcd.com/538/)


====

[HOME](README.md) | [Subreddit](https://www.reddit.com/r/i2p/)
---- | ----
