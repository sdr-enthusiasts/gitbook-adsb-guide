# Why Docker?

Mainly for **Isolation**. Dependencies or settings within a container will not affect any installations or configurations on the host computer, or on any other containers that may be running. By using separate containers for different parts of the ADS-B reception/decode/submission processes, it means the multiple types and versions of software used for each process will not interfere with each other. Bringing online the ability to feed your ADS-B data to another service becomes very simple - just starting another container without having to worry about software conflicts.

Additionally, well crafted containers that take all of their configuration options via environment variables make it really easy to have a single file on the host system that contains all of the settings - instead of having to hunt down various files in various places across the filesystems - sometimes even different places, depending on the host OS or how a specific component was installed. Using Docker Compose allows a very centralized and easy to manage approach to setting up ADS-B - hopefully making it easier for people with any level of skill around computers.

Furthermore, the machine running ADS-B containers can also function as a Plex Server, an OwnCloud Server, a VM Host etc, meaning that you don't need a separate machine just for feeding ADSB data. This means there's less equipment to break and less power used.

