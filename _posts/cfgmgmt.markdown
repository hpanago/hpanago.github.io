Seriously no links, all I got is https://cfgmgmtcamp.eu

### Talk 1: Puppet's Type & Provider Module Extraction Saga

- Generate module with PDK
- Get code from puppet repository
- Build the module and verify it works
- Create a CI pipeline (Jenkins right now at Puppet)
- Documentation -> Puppet Strings
- Release to the forge
- Re-package into puppet-agent
- Remove old code from puppet (now the module does what the old code did)
- On Puppet 6 make sure that the modules you are using are present,
and repackaged to exist on your master
- puppet-agent can go as back as 4.10 (to be compatible with puppet server 6)

### Talk 2: Automating Puppet certificates Renewal

- we already did this at GRNet, perfectly, internal Phab tasks hav eit all
  - We could blog post it
  - we could also use a uppet CA certificate check

talk notes:
- use exit codes in your autosign script
  - puppet masters (CA?) keep the exit code in their logs
- If your puppet masters are in docker containers you could have
specific values injected in your certificates to deploy a secure autosigning
- One role to be associated with one node
- Facts are not inhereted securely
 -  *try FACTER_role = foof*
 - you can make puppet believe wahtever you want(if it is based on facts)
 - When you "clean" your certificates, do you also revoke them?
 
 ### Talk 3: Drilling down a software bug: lessons about observability, monitoring, automation and good practices
 
 Basic takeways
 
 - Collect logs (and structure them to ease your debugging)
 - Get metrics to find out when your app chashes, and what is the state of the rest of the env atm
 - Document your changes
 - Use JXM metrics (especially for Kafka)
 - Distributed tracing (microservices)
 - Zookeeper:
    - can have concorent reads but only sequantial writes
    - master nodes decide for each write
  - Monitor trends
  - Predict scale
  
### Talk 3: Centralized Logging Patterns

- Not ELK Stack anymore -> Elastic Stack
- shipper (beats - logstash) -> parser -> elastic -> kibana
- Log appenders
  - PHP monolog
  - Python: structlog
  - .NET: Nlog
  - JANA: Logback

1. Parse: Take the log files of your applications and extract the relevant pieces of information.
  - MetricBeat -> OS logs, FileBeat -> simple file parsing and shipping
  - Java errors with more that one line get split and get parsed as standalone lines
      - use multine -> pattern = where mutiline_regex
2. Send: Add a log appender to send out your events directly without persisting them to a log file.

3. Structure: Write your events in a structured file, which you can then centralize.

optional: Containerize: Keep track of short lived containers and configure their logging correctly.
Orchestrate: Stay on top of your logs even when services are short lived and dynamically allocated on Kubernetes.

- Use Kibana's Grok debugger to better structure your events
- Structure you logs no matter what
  - use json format (e.g. "the app produces logs in json format"), which removes logstash from the stack
  - can't do it? Dehydrate your logs before elasticsearch gets them
- We can use all the metadata we can get


### Talk 4: Observability Scalability

- Nagios is legacy monitoring
- Metrics based alerting (we care if the app is down not the node, yes they can relate, I know)
- BigGraphite (in house project, fork of Graphite, adapted for Cassandra)
- Centralized metrics
- Centralized logs ( see tailon)
- His team focused on collecting the metrics and did not get involved in alerting
