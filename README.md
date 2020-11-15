# Ansible Infra Automation
Ansible code for Infra Automations.

### Python Jinja2 script

```python
from jinja2 import Template

template = """hostname {{ hostname }}

no ip domain lookup
ip domain name local.lab
ip name-server {{ name_server_pri }}
ip name-server {{ name_server_sec }}

ntp server {{ ntp_server_pri }} prefer
ntp server {{ ntp_server_sec }}"""

data = {
    "hostname": "core-sw-waw-01",
    "name_server_pri": "1.1.1.1",
    "name_server_sec": "8.8.8.8",
    "ntp_server_pri": "0.pool.ntp.org",
    "ntp_server_sec": "1.pool.ntp.org",
}

j2_template = Template(template)

print(j2_template.render(data))
```

### Undefined variable exception
```python
from jinja2 import Template, StrictUndefined

template = "Device {{ name }} is a {{ type }} located in the {{ site }} datacenter."

data = {
    "name": "waw-rtr-core-01",
    "site": "warsaw-01",
}

j2_template = Template(template, undefined=StrictUndefined)
```

### Jinja2 for loop
```python
{% for intf in interfaces -%}
interface {{ intf }}
 description {{ interfaces[intf].description }}
 ip address {{ interfaces[intf].ipv4_address }}
{% endfor %}
```

### Jinja2 Alternate for dictinary variable
```python
{% for iname, idata in interfaces.items() -%}
interface {{ iname }}
 description {{ idata.description }}
 ip address {{ idata.ipv4_address }}
{% endfor %}
```

### If...else condition 
```jinja2
hostname {{ hostname }}
ip routing

{% for intf, idata in interfaces.items() -%}
interface {{ intf }}
  ip address {{ idata.ip }}/{{ idata.mask }}
{%- endfor %}

{% if routing_protocol == 'bgp' -%}
router bgp {{ bgp.as }}
  router-id {{ interfaces.Loopback0.ip }}
  network {{ interfaces.Loopback0.ip }}/{{ interfaces.Loopback0.mask }}
{%- elif routing_protocol == 'ospf' -%}
router ospf {{ ospf.pid }}
  router-id {{ interfaces.Loopback0.ip }}
  network {{ interfaces.Loopback0.ip }}/{{ interfaces.Loopback0.mask }} area 0
{%- else -%}
  ip route 0.0.0.0/0 {{ default_nh }}
{%- endif %}
```

### Logical operation 
```jinja2
{% if x and y -%}
Both x and y are True. x: {{ x }}, y: {{ y }}
{%- endif %}

{% if x or z -%}
At least one of x and z is True. x: {{ x }}, z: {{ z }}
{%- endif %}

{% if not z -%}
We see that z is not True. z: {{ z }}
{%- endif %}
```
### Test variable type
```jinja2
{{ hostname }} is an iterable: {{ hostname is iterable }}
{{ hostname }} is a sequence: {{ hostname is sequence }}
{{ hostname }} is a string: {{ hostname is string }}

{{ eos_ver }} is a number: {{ eos_ver is number }}
{{ eos_ver }} is an integer: {{ eos_ver is integer }}
{{ eos_ver }} is a float: {{ eos_ver is float }}

{{ dns_servers }} is a mapping: {{ dns_servers is mapping }}
```

### Loop with conidition 
```jinja2
{% for intf, idata in interfaces.items() if idata.ipv4_address is defined -%}
{{ intf }} - {{ idata.description }}: {{ idata.ipv4_address }}
{% endfor %}
```

### In operator
```jinja2
    {% if 'Loopback0' in interfaces -%}
```

### Template filtering

1. capitalize
1. join(" ")
1. center
1. default("10")
1. dictsort(by=value, reverse=true)
1. float 
1. groupby(attribute='vlan')
1. int
1. map('lower') | join(", ")
1. {% for as_no in as_numbers| reject('gt', 64495) %}
1. {% for intf in interfaces | rejectattr('mode', 'eq', 'switched') %}
1. {% for as_no in as_numbers| select('gt', 64495) %}
1. {{ interfaces | tojson(indent=2) }}
1. unique 

### Macros function in Jinja2
```jinja2
{% macro def_if_desc(if_role) -%}
Unused port, dedicated to {{ if_role }} devices
{%- endmacro -%}

{% for intf in interfaces -%}
interface {{ intf.name }}
  description {{ def_if_desc(intf.role) }}
{% endfor -%}
```

### Macros in separate file
```jinja2
{% import 'ip_funcs.j2' as ipfn -%}
```

### Macros passing argument
* varargs  -- list
* kwargs -- dictinary kwargs.item()

```jinja2
{% macro set_access_vlan() -%}
{% for intf, vid in kwargs.items() -%}
interface {{ intf }}
  switchport
  switchport mode access
  switchport access vlan {{ vid }}
{% endfor -%}
{%- endmacro -%}

{{ set_access_vlan(Ethernet10=10, Ethernet15=15, Ethernet20=20) }}
```