# Termux-Applications-Installation
A repository containing Wiki/scripts on installation of certain app on native termux.

# Purpose of Repository
While most applications work out of box on PRoot (haven't tested on CHRoot), almost all break on native Termux. This can be explained on the [Termux Wiki](https://wiki.termux.com/wiki/Differences_from_Linux). Native Termux is in a perfect spot between CHRoot and PRoot; the performance is near native if not native such as CHRoot, and does not require root privileges like PRoot. However, the application support is very limited compared to either.

The performance hit on PRoot is really terrible, making even desktop acceleration from CPU laggy while native Termux has very smooth desktop acceleration. While you won't get the patched Adreno driver on native Termux, general usability of Termux is significantly higher than that of PRoot (less laggy desktop), and the patched Adreno driver is buggy on many applications (Chrome does not recognize it as GPU), and in general, many applications do not take use of GPU acceleration (VSCode).

The goal here is to get a significant amount of packages that don't run well/break on Termux.

# Testing Device
Most instructions will be testing on an modern Android device [Samsung Galaxy Tablet S8+]. Particularly Important Features:
- Android 14 with disabled phantom process killing
- Adreno 730 using Zink
- 8GB RAM + 128GB SSD
