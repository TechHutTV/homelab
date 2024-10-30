# Awesome Homelab

This page in the homelab repoistory is a way to feature all of the apps that I have running! While the purpose of the directory is to give a home of all the apps that don't need their own guides and resources, it will still feature everything I run on my machines. Formatting of the readme inspired by [awesome-selfhosted](https://github.com/awesome-selfhosted/awesome-selfhosted).

**This is a work in progress.**

## Automation
* MeTube - Web GUI for youtube-dl, with playlist support, quaility options, and direct downloads. Using this on my Unriad machine, with my projects folder exposed, as the main use case is gathering clips for content creation. [[official repo]](https://github.com/alexta69/metube)

## Business and CRM
Tools I use to run my small media and content product business. 

* Odoo - Odoo is a suite of business apps that covers accounting, CRM, eCommerce, manufacturing, and more. I'm just begining the impletmentation process of this tool as were switching away from Quickbooks. This is mostly open source while some apps require a license even if hosted locally. [[official repo]](https://github.com/odoo/odoo) [[proxy manager config] [[proxy help]](https://github.com/odoo/odoo/issues/104597)
](https://www.odoo.com/forum/help-1/how-to-configure-nginx-proxy-manager-for-odoo-16-245339)

## Dashboards

Dashboards give a graphic user interface to organize, monitorm and view your services or dash.

* [Homearr](https://www.youtube.com/watch?v=A6vcTIzp_Ww) - Sleek, modern dashboard with many integrations and web-based config. Using this as my browser homepage giving a nice overview of everything I need. Currently, I have this setup using on my Servarr Proxmox container using their [compose configuration](https://homarr.dev/docs/getting-started/installation#-docker-compose) in it's own stack, setting the directory to ```/docker/homearr/data:/data```, etc. [[official website]](https://homarr.dev/) [[nzbget fix]](https://github.com/ajnart/homarr/issues/559#issuecomment-1377959472)

## DNS and Ad Blocking

*Pi-hole - The Pi-hole® is a DNS sinkhole that protects your devices from unwanted content without installing any client-side software. My current solution as I am testing out some of the advanced features. [[video]](https://youtu.be/YqaDnnREqI8)

*AdGuard Home - AdGuard Home is a network-wide software for blocking ads & tracking. This is generally my favorite solution  as it has a super clean, easy to use web ui with just enough features for the single use case. [[video]](https://youtu.be/xtMFcVx3cHU)

