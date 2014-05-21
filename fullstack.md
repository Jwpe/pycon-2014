
# Full-Stack Development

## Intro

Topics:

- Semi-typical web app
- Dev, staging, production, deploy
- Helpful Python libs

Not:

- Massively scalable web apps

## Parts of a stack

- OS
- Web server
- Database
- Application language

### Logging

- Log files on machine
- Bigger apps need to aggregate logs across servers
    - Loggly

### Version Control

- git
- svn

### SMTP (Email) Server

- PostmarkApp
- Sendgrid
- Amazon SES

### Async Tasks

- Usually store task queue on Redis or RabbitMQ
- Celery (woo)

### Exception Handling

- Email
- Sentry

### Cache

- Reduce network calls to DB
- Memcache
- Redis
- Varnish

### Monitoring

- Performance, uptime
- New Relic
- Nagios

### Load Balancer

- Can scale multiple servers

### Database replication

- Master -> Slave
- Read-only slaves for reads, write to master

## System of Systems

- Can't edit code in production, so we need multiple environments

### Dev Environment

- Want your environment to be as close as possible to your prod database
- Vagrant
- Venv
- Docker?
- Might need automation to configure these


### Deploy

- Must deploy across multiple applications simultaneously
- Docker, Chef, Puppet, Ansible, SaltStack

### Automated Testing

- Should know your tests are passing before code goes to prod
- TravisCI
- CircleCI

### Staging

- A centralised environment to look at shared code

### Deploying to Separate Codebases

- Smaller codebases are more maintainable - e.g frontend, backend separation (We should do this)

## Hosting

### PaaS

- Heroku
- ELB

## Resources

- github.com/heddle317

## Takeaways

- Basic pieces of a full stack
- What do these pieces look like in different envs?
- Resources