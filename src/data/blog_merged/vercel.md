---
description: Change your DNS settings in Cloudflare to solve this problem
pubDatetime: 2021-09-11
tags:
- Others
title: Vercel Pages Blocked in China
---

Change your A record from

<aside>
📄 76.76.21.21

</aside>

to

<aside>
📄 76.223.126.88

</aside>

Change your CNAME record from

<aside>
📄 [cname.vercel-dns.com](http://cname.vercel-dns.com/)

</aside>

to

<aside>
📄 [cname-china.vercel-dns.com](http://cname-china.vercel-dns.com/)

</aside>

However, I recommend you to use Cloudflare Pages to deploy your website.

Read more here: [deploy-a-hugo-site](https://developers.cloudflare.com/pages/framework-guides/deploy-a-hugo-site)

<aside>
📓 My Chinese site: [blog.ricli.ml](http://blog.ricli.ml)

</aside>