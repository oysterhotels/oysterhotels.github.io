---
title: How to build a 40TB file server
author: Anton
layout: post
permalink: /how-to-build-a-40tb-file-server/
dsq_thread_id:
  - 526878583
categories:
  - Uncategorized
---
The one most valuable asset at <a title="The Hotel Tell-All" href="http://www.oyster.com/" target="_blank">Oyster.com</a> is our photo collection. Take away the intellectual property and what&#8217;s left is, essentially, markup (with a bit of backend to snazz it up.) So we need a solid backup solution for the original high-res photos. The old servers were about to run out of capacity and their slightly outdated specs did not make transferring huge datasets any easier or faster. Thinking a few months ahead, we were looking at a 40TB data set. In strict accordance with <a title="KISS OOM" href="http://techcrunch.com/2009/04/28/keep-it-simple-stupid/" target="_blank">KISS</a> methodology, we opted against <a title="Linear Tape-Open format" href="http://www.lto.org/" target="_blank">LTO</a> and <a title="Simple Storage Service" href="http://aws.amazon.com/s3/" target="_blank">S3</a>, and decided to build a big BOX. (For starters, 40TB on S3 costs around $60,000 annually. The components to build the Box &#8212; about 1/10<sup>th</sup> of that.)

<div id="attachment_565" class="wp-caption alignleft" style="width: 300px">
  <a href="http://tech.oyster.com/wp-content/uploads/2011/12/areca18822.jpg"><img class="size-medium wp-image-565 " title="Areca 1882ix-24 RAID Controller" src="http://tech.oyster.com/wp-content/uploads/2011/12/areca18822-300x182.jpg" alt="Areca 1882ix-24 RAID Controller" width="300" height="182" /></a> 
  
  <p class="wp-caption-text">
    Areca 1882ix-24 RAID Controller
  </p>
</div>

Coincidentally, a great new product was just about to hit the market, reinforcing our decision with its timely relevance &#8212; the dual-core [Areca ARC-1882ix][1] RAID Host Bus Adapter, which comes with an on-board DDR3 SDRAM socket with up to 4GB chip support. Since we already opted for RAID Level 6 (striped, distributed parity&#8211;error checking, tolerates two disk failures) and dual-core <a title="Raid-On-Chip" href="http://www.eetimes.com/electronics-products/other/4085545/RAID-on-Chip-controller-brings-SAS-storage-to-the-masses" target="_blank">RAID-On-Chip</a> means it processes two streams of parity calculations simultaneously &#8212; it seemed ideal.

The first challenge in putting together the big box was getting internal SAS connectors properly seated into the backplane adaptor sockets, the bottom few being especially cumbersome to reach. Thankfully, our hardware technicians&#8217; exceptional manual dexterity rendered having to disassemble the internal fan panel frame unnecessary.

<div id="attachment_577" class="wp-caption alignright" style="width: 300px">
  <a href="http://tech.oyster.com/wp-content/uploads/2011/12/sff80873.jpg"><img class="size-medium wp-image-577 " title="Internal mini-SAS connectors (SFF-8087)" src="http://tech.oyster.com/wp-content/uploads/2011/12/sff80873-300x199.jpg" alt="Internal mini-SAS connectors (SFF-8087)" width="300" height="199" /></a> 
  
  <p class="wp-caption-text">
    Internal mini-SAS connectors (SFF-8087)
  </p>
</div>

The <a title="BIG BOX" href="http://www.norcotek.com/RPC-4224.php" target="_blank">housing assembly</a> comes with six individual backplanes, each accommodating four SAS or SATA disks. Each backplane is secured to the drive bay assembly with three thumbscrews, their shape and material designed to fall within the required torque range when screwed on &#8220;as tight as possible.&#8221; As we found out the hard way, it is absolutely critical to ensure that each of the backplane cards is seated &#8216;full snug&#8217; in the slot and secured dead tight with the thumbscrews. A shoddy connection is not always immediately obvious, it turns out. We observed intermittent timeouts on a particular drive bay as well as degraded overall system performance caused by one of the backplane cards not having been secured quite tight enough &#8212; however, the array was *still functional*, making troubleshooting an opaque nightmare.

<div id="attachment_580" class="wp-caption alignleft" style="width: 300px">
  <a href="http://tech.oyster.com/wp-content/uploads/2011/12/backplane3.jpg"><img class="size-medium wp-image-580 " title="One of the six backplane cards" src="http://tech.oyster.com/wp-content/uploads/2011/12/backplane3-300x198.jpg" alt="One of the six backplane cards" width="300" height="198" /></a> 
  
  <p class="wp-caption-text">
    One of the six backplane cards
  </p>
</div>

One of the most important differences between this system and your run-of-the-mill high performance enterprise server with a couple of hard disks is the addition of: six back-plane cards, one 24-channel raid controller, 24 hard disks, and internal connectors &#8212; all creating a new potential point of failure (at least 37 additional ones). Every single component&#8217;s installation must strictly conform to spec, as the delicately balanced system immediately amplifies any fractional deviations exponentially, resulting in problems persisting for hours, days, and weeks, and many more lost megabytes per second.

If the configuration of all components is optimized, the small individual gains add up to a significant performance boost. At the risk of stating the obvious, things are much less likely to go wrong in a stable, fine-tuned system which performs at max capacity.

<div id="attachment_583" class="wp-caption alignright" style="width: 300px">
  <a href="http://tech.oyster.com/wp-content/uploads/2011/12/bigBOX22.jpg"><img class="size-medium wp-image-583 " title="Big Box with 24 hot-swappable drive bays" src="http://tech.oyster.com/wp-content/uploads/2011/12/bigBOX22-300x104.jpg" alt="Big Box with 24 hot-swappable drive bays" width="300" height="104" /></a> 
  
  <p class="wp-caption-text">
    Big Box with 24 hot-swappable drive bays
  </p>
</div>

One of those things is aligning the physical array dimensions with the file system&#8217;s allocation units, in our case with a stripe size of 64K (since most of our image files are relatively large) on 512 byte blocks, we format using also 64K cluster (&#8220;allocation&#8221;) size. It eliminates the RAID on-board logic overhead of having to keep the logical disk synced with the physical.

A few important things regarding the driver must be mentioned. Windows operating systems inherited their native SAS/SATA RAID controller driver framework from SCSI technology, (SCSIport) and they have several serious drawbacks. I stumbled upon an interesting investigative [ white paper][2] which goes into great detail about these issues. The preferred driver for modern SATA/SAS cards is the STORport driver, developed by the manufacturer’s consortium in response to the inadequate state of native drivers, which inherited limitations of the SCSI protocol. The STORport driver is not certified by Microsoft, therefore the OS by default installs the inferior SCSIport driver. Switching to the STORport driver visibly improved stability and performance during the project&#8217;s earliest stages, instantly bumping write speed by several dozen MB/sec.

<div id="attachment_592" class="wp-caption alignleft" style="width: 300px">
  <a href="http://tech.oyster.com/wp-content/uploads/2011/12/SMOKIN_RAID6_WRITES1.jpg"><img class="size-medium wp-image-592 " title="Gig-E is the bottleneck" src="http://tech.oyster.com/wp-content/uploads/2011/12/SMOKIN_RAID6_WRITES1-300x118.jpg" alt="Gig-E is the bottleneck" width="300" height="118" /></a> 
  
  <p class="wp-caption-text">
    Gig-E is the bottleneck
  </p>
</div>

Having spent some extra time on research,  fine-tuning, and optimizing the new server, we were glad to find that the gigabit network had became the bottleneck, rather than the all-too-commonly disappointing I/O. By aggregating both on-board gigabit network interfaces we can expect transfer rates of 200MB/s (the disks are 3Gbps), which is great for us to maintain a light, low-maintenance incremental backup. Another big advantage of this system in terms of capacity scaling is the external SAS connector which can accommodate another external box of disks to expand our array into. While not the fastest solution possible, it strikes a great balance between performance, value, and reliability (redundancy), which is exactly what we we are looking for.

 [1]: http://www.areca.us/products/1882.htm
 [2]: http://download.microsoft.com/download/5/6/6/5664b85a-ad06-45ec-979e-ec4887d715eb/Storport.doc