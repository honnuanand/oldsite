---
layout: post
title: Cleanup Azure PCF install using BBL
date: '2018-08-09T23:09:00.002-07:00'
author: Anand Rao
tags:
- Pivotal Cloud Foundry
- PCF
- Cloud Foundry
# modified_time: '2018-08-09T23:18:06.053-07:00'
# thumbnail: https://4.bp.blogspot.com/-TNuPZhBtRPo/W20roXa3ehI/AAAAAAAAGQE/mOvrp5-sEv88KBkXVOmgSzq91w6G3tbDwCLcBGAs/s72-c/terraform.state.delete.png
# blogger_id: tag:blogger.com,1999:blog-3696969976504014907.post-6812316351514119746
# blogger_orig_url: https://blog.clue2solve.io/2018/08/cleanup-azure-pcf-install-using-bbl.html
---

The Power of using automation and starting to look at the <a href="https://content.pivotal.io/blog/product-all-the-things">platform as a product</a> is that you can wipe out everything and be sure that you can get back to the same state you were in.
To Test out that Theory, I decided to use the bbl tool to blow the platform off and then rerun the pipeline and get back to this state. Thanks to my good friend Zack Bergquist for pointing me in this direction. `bbl` has this very powerful ( I mean, be careful when you run it ) feature that connects to your IAAS using your credentials provided.

A quick execution sample is here:
{% highlight bash %}
$ anandrao at Anands-MBP in ~/pivotal/repos/bbl-az/bbl/n$ bbl cleanup-leftovers
[Resource Group: araoc2sterraform] Delete? (y/N): n
[Resource Group: bbl-env-caspian-2018-08-07t01-09z-bosh] Delete? (y/N): n
[Resource Group: bbl-env-ontario-2018-07-31t05-15z-bosh] Delete? (y/N): n
[Resource Group: c2s] Delete? (y/N): y
[Resource Group: cloud-shell-storage-westus] Delete? (y/N): n
[Resource Group: dns] Delete? (y/N): n
[Resource Group: jump] Delete? (y/N): n
[Resource Group: c2s] Deleting...
[Resource Group: c2s] Deleted!
$ anandrao at Anands-MBP in ~/pivotal/repos/bbl-az/bbl
{% endhighlight %}

Notice , I did not delete my DNS zones, bbl and other groups. The only Group I deleted was "c2s" the Resource group that had my Full PCF install.
Along with that , I will have to delete the terraform.state file on the container created for this.
You could do that on command line via the azure cli or the UI as below Job. It creates a fresh state file. 

Basically follow the instructions on the [part 3()] of the blog series to get you there.

Azure" Blog Series :
#####    1. Installing PCF 2.X on Azure using Concourse - Part - 1 , Bosh Boot loader and bosh
#####    2. Installing PCF 2.X on Azure using Concourse - Part - 2 , Concourse using bosh
#####    3. Installing PCF 2.X on Azure using Concourse - Part - 3 ,Installing Pivotal Cloud Foundry using a Pipeline
#####    4. Domain Name System (DNS) Delegation with Azure DNS Zones
#####    5. Cleanup Azure PCF install using BBL