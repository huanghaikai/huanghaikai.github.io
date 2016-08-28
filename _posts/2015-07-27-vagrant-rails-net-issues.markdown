---
layout: post
title:  "Rails 4.2, Vagrant and ... Problems!"
date:   2015-07-27 22:39:00
categories: recurse-center
comments_enabled: true
---

I started a new personal project. I set up [Vagrant][1] so I can have the "same" development environment across all computers.

The project uses [Ruby on Rails][2], so I set up the line below in the `Vagrantfile`.

```Ruby
config.vm.network "forwarded_port", guest: 3000, host: 3000
```
It means forward port 3000 from the guest as if it was 3000 on the host.

I started up the virtual machine and ran `rails server`. From the host I fired up Safari open address `http://localhost:3000`. I expected to see my app, but what I saw was:

![CannotOpen](/public/CannotOpen.jpg "Can`t connect")

Of course, browsing on the Internet for possible solutions was the next step. All the effort I spent Googling was in vain. In part because I looked for everything in the log which may seem like a problem, like:

    default: The guest additions on this VM do not match the installed version of
    default: VirtualBox! In most cases this is fine, but in rare cases it can
    default: prevent things such as shared folders from working properly. If you see
    default: shared folder errors, please make sure the guest additions within the
    default: virtual machine match the version of VirtualBox you have installed on
    default: your host and reload your VM.
    default:
    default: Guest Additions Version: 4.3.10
    default: VirtualBox Version: 5.0

**Totally misleading** and the wrong approach for this particular problem.

##The analysis##

**How does `vagrant ssh` work but `rails server` not? Both rely on port forwarding** This was the starting point question.

I tried listening with [nc][3] (netcat) on port 3000 and making a requests from the host. Unfortunately no response again:

```Bash
nc -l localhost 3000
```

No matter what I sent to port 3000 on the guest, it was not reached.

So nothing to do with Rails, right? **No, but yes!** I don't know why, but I read the manpage for `nc`, and found out that `-p` is for ports, by trying `nc -l -p 3000` I could reach the guest from the host on the desired port.

```Bash
vagrant@vagrant-ubuntu-trusty-64:~$ nc -l -p 3000
GET / HTTP/1.1
Host: localhost:3000
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
If-None-Match: W/"b56dd5f9363ed0f7bd4d11c36d9471dd"
Cookie: _ga=GA1.1.1880364256.1438043513
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_4) AppleWebKit/600.7.12 (KHTML, like Gecko) Version/8.0.7 Safari/600.7.12
Accept-Language: en-us
Accept-Encoding: gzip, deflate
Connection: keep-alive
```

**But what does Rails have to do with it?**

Reading the change log from Rails 4.2 [here][4]. We can find the text below:

"Due to a change in Rack, rails server now listens on localhost instead of 0.0.0.0 by default "

##What I learned##

127.0.0.1 (or localhost) **is not the same** as 0.0.0.0. 

The ip address **0.0.0.0** means **all IP addresses on the local machine**. It includes all interfaces, including the virtual interface used by virtualbox.

**127.0.0.1** is the "loopback connection", it is **only be available from the same host/system**. In this case, it was the guest virtual machine. This was the reason why I could not reach from the host. The guest is a different host/system than the guest. 

##The solution##

Make rails listed on 0.0.0.0. This can be achieved by running rails server this way: `bin/rails server -b 0.0.0.0`, or by changing file `config/boot.rb` to include:

```Ruby
require 'rails/commands/server'

module Rails
  class Server
    new_defaults = Module.new do
      def default_options        
        default_host = Rails.env == 'development' ? '0.0.0.0' : '127.0.0.1'
        super.merge( Host: default_host )
      end
    end

    # Note: Module#prepend requires Ruby 2.0 or later
    prepend new_defaults
  end
end
```
If you make this change, you can just use `rails server` as usual.

[1]: https://www.vagrantup.com
[2]: http://rubyonrails.org
[3]: http://netcat.sourceforge.net
[4]: http://guides.rubyonrails.org/4_2_release_notes.html#default-host-for-rails-server