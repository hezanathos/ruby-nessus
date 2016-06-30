# Ruby-Nessus

## Description

Ruby-Nessus is a ruby interface for the popular Nessus vulnerability scanner. Ruby-Nessus aims to deliver an easy yet powerful interface for interacting and manipulating Nessus scan results and configurations. Ruby-Nessus currently supports both version 1.0 and 2.0 of the .nessus file format. Please remember to submit bugs and request features if needed.

More Information:
* Documentation: http://rdoc.info/projects/mephux/ruby-nessus
* More: http://www.packetport.net

## Install

  ```sudo gem install ruby-nessus```

## Usage & Examples

The below example illustrates how easy it really is to iterate over result data.
```ruby  
  require 'rubygems'
  require 'ruby-nessus'

  RubyNessus::Parse.new("example_v1.nessus", :version => 1) do |scan|
  # OR: RubyNessus::Parse.new("example_v2.nessus") do |scan|   <-- Ruby-Nessus will figured out the correct Nessus file version.
  
    puts scan.title                     # The Nessus Report Title.
    puts scan.host_count                # Host Count.
    puts scan.unique_ports              # All Unique Ports Seen.

    scan.each_host do |host|
      next if host.event_count.zero?    # Next Host If Event Count Is Zero.
      puts host.hostname                # The HostName For The Current Host.
      puts host.event_count             # The Event Count For The Current Host.

      host.each_event do |event|
        next if event.severity.medium?  # Next Event Is The Event Severity Is Low. (supports high? medium? low?)
        puts event.name if event.name   # The Event Name If Not Blank.
        puts event.port                 # The Event Port. (supports .number, .protocol and .service)
        puts event.severity.in_words    # The Current Event Severity In Words. i.e "High Severity"
        puts event.plugin_id            # The Nessus Plugin ID.
        puts event.data if event.data   # Raw Nessus Plugin Output Data.
      end
    end
  end
```

You also have the ability to search for particular hostnames. In the near future I plan to add the ability to pass the hosts block a hash of options for more complex searches.
```ruby
  scan.find_by_hostname("127.0.0.1") do |host|

    puts host.scan_start_time
    puts host.scan_stop_time
    puts host.scan_runtime

    host.high_severity_events do |event|
      puts event.severity.in_words
      puts event.port
      puts event.data if event.data
    end

  end
```
There are a bunch of convenient methods (maybe more then needed) added to make reporting a bit easier to produce quickly from a raw scan file. If you do not pass :version as an option it will default to the 2.0 .nessus schema.
```ruby
  RubyNessus::Parse.new("example_v2.nessus") do |scan|

    puts scan.event_percentage_for('low', true) #=> 8%

    puts scan.high_severity_count           # High Severity Event Count
    puts scan.medium_severity_count         # Medium Severity Event Count
    puts scan.low_severity_count            # Low Severity Event Count
    puts scan.open_ports_count              # Open Port Count

    puts scan.total_event_count #=> 3411    # Total Event Count
    puts scan.hosts.count #=> 12

    
    scan.each_host do |host|
      puts host.hostname
      puts host.event_percentage_for('low', true)
      puts host.tcp_count #=> tcp, icmp, udp supported.
    
      host.each_event do |event|
        next if event.informational?
        
        puts event.severity.in_words
        puts event.synopsis
        puts event.description
        puts event.solution
        puts event.output
        puts event.risk
        
      end
  
    end

  end
```
Ruby-Nessus also ships with a POC CLI application for the lib called 'recess':
```  
  Recess 0.1.1
  usage: recess FILE [OPTIONS]
      -f, --file FILE                  The .nessus file to parse.
      -h, --help                       This help summary page.
      -v, --version                    Recess Version.
```
Below is example output generated by recess:
```  
  $> recess examples/example_v2.nessus 
  Recess - Ruby-Nessus CLI
  Version: 0.1.1

  -> SCAN Metadata: 

  	Scan Title: Ruby-Nessus
  	Policy Title: Ruby-Nessus

  -> SCAN Statistics: 

  	Host Count: 2
  	Open Port Count: 51
  	TCP Count: 38
  	UDP Count: 11
  	ICMP Count: 1

  -> EVENT Statistics: 

  	Informational Severity Count: 19
  	Low Severity Count: 47
  	Medium Severity Count: 3
  	High Severity Count: 0
  	Total Event Count: 50


  	Low Event Percentage: 94
  	Medium Event Percentage: 6
  	High Event Percentage: 0

  -> HOSTS: 

  	Hostname: snorby.org
  		- IP Address:: 173.45.230.150
  		- Informational Count: 12
  		- Low Count: 34
  		- Medium Count: 1
  		- High Count: 0

  	Hostname: scanme.insecure.org
  		- IP Address:: 64.13.134.52
  		- Informational Count: 7
  		- Low Count: 13
  		- Medium Count: 2
  		- High Count: 0
```  
## Requirements
* Ruby 1.8 or 1.9
* Nokogiri http://github.com/tenderlove/nokogiri

## Todo
* Add The Ability to parse the scan configuration and plugin options.
* Building XML (.nessus) files configurations
* Add Support For NBE File Formats.

## Note on Patches & Pull Requests 
* Fork the project.
* Make your feature addition or bug fix.
* Add tests for it. This is important so I don't break it in a
  future version unintentionally.
* Commit, do not mess with rakefile, version, or history.
  (if you want to have your own version, that is fine but bump version in a commit by itself I can ignore when I pull)
* Send me a pull request. Bonus points for topic branches.

## Copyright

Copyright (c) 2009 Dustin Willis Webber. See LICENSE for details.
