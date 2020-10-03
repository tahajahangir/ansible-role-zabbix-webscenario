# ansible-role-zabbix-webscenario

Ansible role which creates or deletes (NO UPDATE) zabbix web scenarios
For each entry in the 'zabbix_web_check_websites' variable with state: present, this role will :

* Add a web scenario
* Add a basic step
* Add a trigger associated to that web scenario

For each entry in the 'zabbix_web_check_websites' variable with state: absent, this role will :
* Delete the web scenario associated to its name
* Delete the trigger associated to it

For any update of an existing web scenario, it must first be deleted (manually or by changing its state to absent, executing the playbook, then changing it back to 'present')

### Zabbix versions
This role was tested and validated with :
* Zabbix 3.2
* Zabbix 3.4
* Zabbix 4.0

### Variables

Here is the list of all variables and their default values :
```yaml

# Authentification to API
zabbix_api_user: api_ansible                     #zabbix user to authenticate with API, must have access to the zabbix_web_check_host
zabbix_api_pass: 'strongpassword'                #zabbix user password
zabbix_url: 'https://zabbix.mycompany.com'       #zabbix URL

# Common variables to all webscenarios
zabbix_web_check_host: "zabbixproxy01"           # default name of the zabbix host to which the scenario will be attached to (must be allready created)
zabbix_web_check_application_name: 'Web Check'   # default application name (category like) (must be already created, and attached to zabbix_web_check_host)
zabbix_web_check_interval: 30                    # default interval of execution of the web scenarios
zabbix_web_check_retries: 5                      # default maximum number of retries
zabbix_web_check_ua: 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/38.0.2125.104 Safari/537.36' #useragent used by zabbix

# Definition of each website to monitor
zabbix_web_check_websites:
    ## Creates a webscenario with one step
    ## step: https://mycompany.com using GET HTTP method, expects 200,301 or 302 as return code, expects string 'Welcome to mycompany website'
    ## The associated trigger will have two tags, display and any_other_tag.
  - name: 'Our Company Website'                # name of the web scenario
    url: 'https://mycompany.com'               # url of the web scenario (with http(s)://)
    state: present                             # state, present (default) or absent
    method: 'GET'                              # http method used, GET or HEAD, GET by default
    status_codes: '200,301,302'                # expected http code returned, 200 by default
    watch_window: '5m'                         # trigger threshold (3-Minute default, #4 or ...)
    required: 'Welcome to mycompany website'   # required string on the page, method MUST be GET for this to work
    auth:                                      # for websites that require basic HTTP authentication
      http_user: 'jim'                                  
      http_password: 'jimspassword'
    trigger_tags:                              #tags to add with the associated trigger
      display: 'yes'
      any_other_tag: 'any_value'

    ## Creates a webscenario with one step, checking http://anotherwebsite.com/ using GET HTTP method, expects 200 as return code
  - name: 'Another website'
    url: 'https://anotherwebsite.com'

    ## Other possible variables to put in zabbix_web_check_websites
    zabbix_host:                                       #attaches the web scenario on another host than zabbix_web_check_host
      name: 'some_other_host'
      hostid: 4567
      applicationid: 1234
    ssl_verify_peer: 'no'                      #for https websites, disables the checking of the matching between the cn and the domain name of the website (default yes)
    ssl_verify_host: 'no'                      #for https websites, disables the checking of the root certificate against trusted store (default yes)

```

### Usage

Clone the repo.
```bash
$ git clone https://github.com/jdelvecchio/ansible-role-zabbix-webscenario
```
Then set vars in your playbook file.

Example playbook file :

```yaml
---
- hosts: 127.0.0.1
  gather_facts: no
  become: no
  roles:
    - role: ansible-role-zabbix-web-scenario
      zabbix_api_user: api_ansible
      zabbix_api_pass: 'strongpassword'
    
      zabbix_url: 'https://zabbix.mycompany.com'
      zabbix_web_check_host: "scasrvzbx01pp"
      zabbix_web_check_application: 1457
      zabbix_web_check_interval: 30
      zabbix_web_check_retries: 5
      zabbix_web_check_ua: 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/38.0.2125.104 Safari/537.36'

      zabbix_web_check_websites:
        - name: 'Our Company Website'
          url: 'https://mycompany.com'
          state: present
          method: 'GET'
          status_codes: '200,301,302'
          required: 'Welcome to mycompany website'
          trigger_tags:
            display: 'yes'
            any_other_tag: 'any_value'

        - name: 'Another website'
          url: 'http://anotherwebsite.com'
          state: present
```
