+++
date = '2021-07-23T13:06:17+01:00'
draft = false
title = 'Startup idea: .nonpublic trusted TLS'
+++

What if we could make the practical use of self-signed TLS certificates a painless experience?

Accessing an intranet website protected with a self-signed TLS certificate is not a smooth experience. The browser does not trust the certificate unless it is installed in the OS trust store or explicitly accepted; bypassing the security warning makes a norm out of dismissing them; developers are discouraged from making TLS a first-class part of their local environment because of these issues. How can we solve this problem and make accessing https://foo.bar.local error-free, with a widely-trusted certificate? And is there a viable business idea in the solution?

Recently I explored the viability of creating a new gTLD, .nonpublic, and becoming a DNS registrar. Users would purchase a zone for a small fee, such as bar.nonpublic. There would be an unusual twist to this TLD: no A, AAAA, CNAME, MX etc. records would be accepted. Only TXT records would be accepted, with the intention being that these would be used to satisfy DNS-01 ACME challenges for the issuance of TLS certificates, e.g. to *.bar.nonpublic.

Intranets would be expected to have their own internal DNS resolvers handling lookups for foo.bar.nonpublic. Using the upstream root nameserver for .nonpublic would result in NXDOMAIN. WHOIS searches would return nothing. The grand idea is that a developer could use a service like Let's Encrypt to obtain a TLS certificate for an internal domain but strictly forbid any information about the internal network being stored in the public internet. Use of .nonpublic would therefore not in itself require egress traffic (except when minting certificates).

A number of design decisions have been briefly touched upon here, which are worth discussing:

* **Zones.** Purchase of a zone like bar.nonpublic for your exclusive use is necessary. One might wonder why that is so: after all, if all record lookups (except TXT) will receive NXDOMAIN, what does it matter if two networks internally know themselves as that zone? The answer is that if I am able to mint a certificate for any given zone, without precondition, then I pose a serious threat to you if I can gain such access to your systems as to poison your internal DNS. In other words, if I could re-point your internal resolution of your foo.bar.nonpublic to my own site, and present a certificate for same, then I've pwned you. For this reason, only one party will be able to own a zone at a given time; 2FA will be required to log in to generate API credentials for programmatic TXT record updates as part of the DNS-01 ACME challenge.
* **Non-resolution of any name to any IP.** The central principle of the .nonpublic gTLD is that it is for use by non-public sites to obtain trusted certificates for their internal endpoints, and nothing else. It is not your intranet's service discovery mechanism. Any name lookups within a network with this TLD should be resolved by an internal nameserver: attempts to use the root nameserver will purposefully fail. This prevents leakage about your internal service topography. Additionally, there will be limits on the number of TXT records per zone at a given time, to prevent abuse.
Reliance upon use of well-known CAs. We're trying to avoid the problem of browsers rejecting self-signed certificates. We therefore want to use certificate authorities which are typically already trusted in the trust stores of mainstream operating systems. Let's Encrypt is the obvious example.
* **Low cost.** A few dollars per year per zone is about right. The mission is to maximally increase the trivial adoption of TLS everywhere, with a focus on internal networks. A low cost means a low barrier to entry.
Careful choice of gTLD. We can't register, for instance, .local because it has special meaning in many systems (though isn't RFC-reserved) and Let's Encrypt won't mint certificates for its subdomains. The idea of choosing .nonpublic is that it clearly indicates the intention of the domain.

## Resources required

A gTLD costs $190k to set up plus $25k/year. Let's pencil in $25k/year for infrastructure costs (root nameservers and bandwidth). Allow a further $360k/year for staffing costs related to employment of people who can run the root nameservers, build and run the registrar portal and API, and generally respond to enquiries. (These numbers are very minimalistic, but still minimally viable.) This totals $600k in expenses. I wouldn't want to charge more than $5/zone/year. More than this hampers accessibility, particularly in oppresive, lower-income countries where the adoption of TLS should be maximally encouraged. Perhaps a reasonable compromise would be $3/zone/year if you will accept a randomly-generated zone (e.g. gs23favv9.nonpublic) and a $10/year/zone "premium" product if you want to specify the zone name. With 10% adoption of the premium product, we can quickly see that we break even at approximately 163k zones. This does not seem crazy.

## Arguments against

* Why not just register foobar.com in Route53 and have a private zone for this purpose? Certainly you can do that, and many people do. But:
* Let's not encourage such vendor lock-in. (Not that you need to use AWS, of course.)
* You lose the (counter-intuitive) feature of guaranteed NXDOMAIN for everything - the feature which forces you not to use remote nameservers for your internal systems.
* Your private zone might quickly morph into a service discovery system requiring dynamic DNS updates.
* Good luck using AWS in Iran etc.
* This line of argument reminds me somewhat of how you can "trivially" replace Dropbox with SFTP, clever filesystem mounting and a VCS. Yes, you are right, but the point is to create a convenient product targeted at solving a particular problem.
* You know that there is a public record of Let's Encrypt certificates, right? You're pointing out that service topographies might be thereby revealed. We avoid this problem by defaulting to randomly-generated zones and recommending use of wildcard certificates, which reveal no useful information.

## Risks

Unfortunately, there are several, serious risks with this idea.

* What if Let's Encrypt decide they don't like the idea and refuse to issue certificates for this gTLD?
* What if GoDaddy, or similar, say "cool idea bro" and replicate it within a week?
* What if someone tries to sue me for wanting to keep my monopoly of registrars for the gTLD?

## Risk Mitigations

Are there ways we can de-risk this as a business idea? What about the following?

* Register nonpublic.co.uk or perhaps nonpub.lc (St. Lucia, if you're wondering). Customers purchase zones like bar.nonpub.lc and get TLS certificates for, say, *.bar.nonpub.lc.
* Run a couple of nameservers for this domain on cheap VPSs and delegate nonpub.lc to them.
* Don't spend thousands on a gTLD, or staff; instead spend a few hundred on building out an MVP myself.
* There is a fatal flaw: Let's Encrypt rate limits. Everyone using this service would present to Let's Encrypt as a subdomain of nonpub.lc, and very quickly exhaust rate limits.

## Conclusion

The above-discussed arguments against, and risks, are actually pretty strong. Registering a name for the purposes of certificate generation, but just never using that domain for anything else, is trivial to do for anyone who sufficiently understands the motivations. Internal DNS setups to point example.com somewhere internal aren't hard. There isn't a mass consumer product in this, as in the case of Dropbox. And there is plenty of scope for this whole plan to be sunk below the waterline should Let's Encrypt object; and it could be replicated in a week by a bigger player.

Therefore, sadly, it seems there isn't a viable startup idea here after all.
