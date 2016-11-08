https://www.vulnhub.com/entry/imf-1,162/

    Welcome to "IMF", my first Boot2Root virtual machine. IMF is a intelligence agency that you must hack to get all flags and ultimately root. The flags start off easy and get harder as you progress. Each flag contains a hint to the next flag. I hope you enjoy this VM and learn something.
    
Import and start VM
Local network 172.16.1.0/24

netdiscover -r 172.16.1.0/24
MAC Vendor Camdus Computer Systems = VM Virtualbox

Found possible target - 172.16.1.76
nmap -sS 172.16.1.76
# port 80

use browser - IMF Webpage. Click around
