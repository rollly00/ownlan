# Ownlan

__Ownlan__ aims to be a simple, concise and useful pentesting LAN poisoning suite, Written in ``Ruby`` and using ``PacketFU`` for reading and sending the packets off the wire. I decided to make this suite of tools mainly due do to the lack of existing tools on Linux, on top of helping me understanding the whole process behind the scene. OwnLan got uniques features, with some exclusives and excitings attacks probably never ever used on a (pentesting) network.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'ownlan'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install ownaln

And require it in your application:

    irb(main):001:0> require 'ownlan'
    => true


## Usage

### Configuration ###
You can pass multiple variables that will be used in the gem.

```ruby
Ownlan.configure do |config|
      config.attack      = 'ntoa'
      config.victim_ip   = '192.168.0.1'
      config.delay       = 1.5
      config.random_mac  = true
      config.interface   = 'eth0'
end
```

You can also pass any of those options inline when loading an instance of Ownlan.

```ruby
ownlan = Ownlan.new(attack: 'ntoa', victim_ip: '192.168.0.1', delay: 1.5, random_mac: true)
```

## Launch Ownlan ##

### In your Ruby code ###

Once configured, you can run your instance of Ownlan with:

```ruby
ownlan.call
```

You are free to implement whatever way of your choice to handle concurrency: you can put this previous line in a `thread` for instance.

### Using the Command Line Interface  ###

```sh
  ownlan --attack client --target-ip 192.168.0.1 --interface eth0 --delay 0
```

Please look at the [cli section](#command-line-interface) for more advanced options.

## Features

**OwnLan** has four features:

1. Disconnect one or several users off the wire
2. Protecting users from those kind of attacks
3. Sending custom ARP + DHCP packets easily
4. Easy ARP packets capture *[Not Implemented]*

### Disconnecting users off the wire

The biggest part of **OwnLan**. It disconnects clients thanks to severals techniques:

- Client side ARP Cache Poisoning (**first duplex**). *The most used and common attack nowadays, the main purpose is to make a MITM attack, but alone (= without IP forwarding), it will disconnect the client. Used by ``TuxCut`` and ``Arpspoof`` . If no MAC Adress is given, yours will be given.*

- Gateway side ARP Cache Poisoning (**second duplex**). *A less known attack and powerful one, used by ``NetCut`` , the principle is to give the gateway a fake correspondancy of the victim MAC Adress to make the later one unreachable. If no MAC Adress is given, yours will be given.*

- Neighbour Table Overflow attack. *I invented this attack, not to be modest. In fact, I should say 'implemented' since, usually, it is the gateway which is attacked (even so, this one attack is very rare), here, we attack the client directly. I don't think anyone has ever thought of this... and it works! The **NTOA** will not insert random MAC adress, but following a scheme, to ensure that 2 same mac adress won't be injected. So, it makes the attack faster. One client or all clients can be specified.*

- Gateway ARP Cache Overflow. *This attack will render the GateWay ARP Cache unusable, and will make a loss of connectivity to all the clients. Some CISCO routers are immuned to this attack though*

- DHCP Lease Spoofing [Not Implemented]. *This attack will spoof DHCP lease (udp) packet by telling the DHCP server 'Hello, I don't use this IP anymore, just disconnect me' . As of today, this attack is extremly rare, difficult to make, and used only by ``Yersinia`` . There is nothing to prevent this attack, after it has been used. Really.


### Command Line Interface

You can also use the provided executable. Simple launch it in accordance to the following scheme:

    ownlan --[options] [sub-options] --[other-option]

- Where [options] are either:
 

        -a, --attack=<s>          Set an attack on a device on the network
                                  * Required: [sub-options]
        -p, --protect=<s>          Protect a device from lan attacks
                                  * Required: [sub-options]
        -b, --broadcast=<s>        Broadcast raw ARP packets to the wire.
                                  * Required Options : victim_ip, victim_mac, source_ip, source_mac
        -c, --capture=<s>          Sniffing ARP packets on the network [Not Implemented]


-  where [sub-options] are either:


        client              Set a First-Duplex disconnection attack (the client is targeted). If no source mac argument, yours will be given (useful for MITM Attacks).
                          * Required options: victim_ip
                          * Falcultative options: random_source_mac , source_mac
        gateway             Set a Second-Duplex disconnection attack (the gateway is targeted). If no source mac argument, yours will be given (useful for MITM Attacks).
                          * Required options: victim_ip
                          * Falcultative options: random_source_mac , source_mac
        ntoa                The client is targeted to get disconnected, using a neighbour table overflow attack. Requires a victim ip.
                          * Required options: victim_ip
                          * Falcultative options: random_source_mac
        fake-ip-conflict    Generate a fake ip conflict to the victim. Can be used along all the others attacks, or alone.
                          * Required options: victim_ip [Not Implemented]

        resynchronize       Resynchronize the Gateway ARP Cache by sending to it continuous healthy correspondancies packets to protect someone or yourself from gateway. Default params are your mac and the gateway's mac. WARNING: If your gateway_mac is the attacker's, this protection won't work. In this case, input the gateway's mac manually.
                          * Optional options: victim_mac , gateway_mac
        stealth             Becomes invisible from network scanners, preventing you from getting targeted. [Not Implemented]
        static              Set a static ARP Cache for the current session. Good against first-duplex ARP Cache Poisoning. [Not Implemented]
        freeze              Reset and Freeze your ARP Cache. Good against NTOAs. [Not Implemented]


        attack. (reveive IP or MAC argument)


- Where  [Other Options] can be:


        -d, --delay=<f>            Set the time lapse delay between each packet (default: 0.5)
        -i, --interface=<s>        Set the network interface which will be used (default: wlan0)
        -r, --random-source-mac    If setted, the used origin addresses will be randomly generated.
        -t, --victim-ip=<s>        Set the ip address of the target.
        -v, --victim-mac=<s>       Set the mac address of the target
        -g, --gateway-ip=<s>       Set the ip adress of the gateway
        -e, --gateway-mac=<s>      Set the mac adress of the gateway. (for Protect only)
        -s, --source-mac=<s>       Set the mac of the source mac address.
        -o, --source-ip=<s>        Set the ip address of the originating packet.
        -n, --version              Print version and exit
        -h, --help                 Show this message




## Versioning

__Ownlan__ follows [Semantic Versioning 2.0](http://semver.org/).

## Contributing

1. Fork it ( https://github.com/shideneyu/ownlan/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request

## Contact

Any question ? Feel free to contact me at `contact(at)sidney.email` .
Any issue ? Open a [ticket](https://github.com/shideneyu/ownlan/issues) !

## License

Copyright (c) 2016 Sidney Sissaoui

Released under the MIT license. See [LICENSE.md](https://github.com/shideneyu/ownlan/blob/master/LICENSE.md) for more details.
