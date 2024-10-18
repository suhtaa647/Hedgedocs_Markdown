---
title: development
tags: 11, 3days-optional, 4days-optional, 5days-optional
---

# development
create a collection first

## jinja filter

### documentation

[Filter plugins --- Ansible Community Documentation](https://docs.ansible.com/ansible/latest/plugins/filter.html)
[Developing plugins --- Ansible Community Documentation](https://docs.ansible.com/ansible/latest/dev_guide/developing_plugins.html#developing-filter-plugins)

#### examples

[ansible/lib/ansible/plugins/filter at devel · ansible/ansible](https://github.com/ansible/ansible/tree/devel/lib/ansible/plugins/filter)

### list all available filter plugins

``` shell
ansible-doc -t filter -l
```

### location

#### configuration

[Ansible Configuration Settings --- Ansible Community Documentation](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#default-filter-plugin-path)

### create folder for filters

``` shell
mkdir ~/code/b1/ansible/plugins/filter
```

### create a working playbook

`~/code/b1/ansible/playbooks/FilterTest.yml`
``` yaml
---
- name: 'Testrolle für JinjaFilter'
  hosts: all
  vars:
    input: 'input, der gefiltert werden soll'
  tasks:
    - name: 'Use the filter'
      ansible.builtin.debug:
        msg: "{{ input }}"
        
```


### implement the filter
`~/code/b1/ansible/plugins/filter/filters.py`

``` python
class FilterModule(object):
    def filters(self):
        return {
            'append_message': self.append_message,
            'uppercase_input': self.uppercase_input,
            'multiply_all_the_spaces': self.multiply_all_the_spaces,
        }

    def append_message(self, input_before_bar):
        '''Append a message to the input.'''
        return input_before_bar + ' - vom Filter hinzugefügt'

    def uppercase_input(self, input_before_bar):
        return input_before_bar.upper()

    #  def multiply_all_the_spaces(self, input_before_bar, amount):
    def multiply_all_the_spaces(self, input_before_bar, amount=2):
        return input_before_bar.replace(" ", " "*amount)
    
```

### documentation
### create files for documentation
multiple filter can be defined in one python-module
but each filter needs it's own documentation-file

#### `~/code/b1/ansible/plugins/filter/append_message.yml`
```yaml=
DOCUMENTATION:
  name: append_message
  version_added: "historical"
  short_description: append an example message to the input string
  description:
    - A defined example message is appended to the input string
  positional: _input
  options:
    _input:
      description: Data to concatenate.
      type: raw
      required: true

EXAMPLES: |
  # in a task
  # msg: "{{ input | append_message }}"

RETURN:
  _value:
    description: the input string concatenated with the appended message
    type: string
```
#### `~/code/b1/ansible/plugins/filter/uppercase_input.yml`
```yaml=
DOCUMENTATION:
  name: uppercase_input
  version_added: "historical"
  short_description: uppercase all chars contained in input
  description:
    - convert all characters from the input to their uppercase equivalent
  positional: _input
  options:
    _input:
      description: Data to convert.
      type: raw
      required: true

EXAMPLES: |
  # in a task
  # msg: "{{ input | uppercase_input }}"

RETURN:
  _value:
    description: the string converted to uppercase
    type: string
```

#### `~/code/b1/ansible/plugins/filter/multiply_all_the_spaces.yml`
```yaml=
DOCUMENTATION:
  name: multiply_all_the_spaces
  version_added: "historical"
  short_description: multiply every found space by the requested amount
  description:
    - every space found in the input string will be substituted by n*spaces
    - where n is given as parameter
    - |
      default value for n is: 2
  positional: _input
  options:
    _input:
      description: Data containing spaces to multiply.
      type: raw
      required: true
    amount:
      description: Factor by which every space is multiplied
      type: int
      default: 2
      required: false

EXAMPLES: |
  # in a task
  # msg: "{{ input | multiply_all_the_spaces()  }}"
  # msg: "{{ input | multiply_all_the_spaces(9) }}"

RETURN:
  _value:
    description: the string with all spaces substituted by n spaces
    type: string
```

#### show documentation for a specific filter
``` shell
ansible-doc -t filter append_message
ansible-doc -t filter uppercase_input
ansible-doc -t filter multiply_all_the_spaces

### after redeploying the collection
ansible-doc -t filter b1.ansible.uppercase_input
ansible-doc -t filter b1.ansible.uppercase_input
ansible-doc -t filter b1.ansible.multiply_all_the_spaces
```

### use the filter

#### playbook
`~/code/b1/ansible/playbooks/FilterTest.yml`


``` yaml
---
- name: 'Testrolle für JinjaFilter'
  hosts: localhost
  vars:
    input: 'input, der gefiltert werden soll'
  tasks:
    - name: 'Use the filter'
      ansible.builtin.debug:
        msg: "{{ input }}"
        # msg: "{{ input | append_message }}"
        # msg: "{{ input | append_message | uppercase_input }}"
        # msg: "{{ input | append_message | uppercase_input | multiply_all_the_spaces    }}"
        # msg: "{{ input | append_message | uppercase_input | multiply_all_the_spaces(1) }}"
        # msg: "{{ input | append_message | uppercase_input | multiply_all_the_spaces(9) }}"

```
#### execution
```shell
ansible-playbook /home/tux/code/b1/ansible/playbooks/FilterTest.yml
```



## module

### INFO

[Module format and documentation --- Ansible Community Documentation](https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_documenting.html)
[Developing modules — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_general.html#creating-an-info-or-a-facts-module)


    A module is a reusable, standalone script that Ansible runs on your behalf, either locally or remotely. 
    Modules interact with your local machine, an API, or a remote system to perform specific tasks. 
    A module provides a defined interface, accepts arguments, and returns information to Ansible by printing a JSON string to stdout before exiting.

### WORKFLOW

1. create directory for modules

``` shell
mkdir -p ~/code/b1/ansible/plugins/modules
```

3. create the module

`~/code/b1/ansible/plugins/modules/hello_world.py`

``` python=
#!/usr/bin/python

# Copyright: (c) 2024, Tux Pinguin tux@b1-systems.de
# GNU General Public License v3.0+ (see COPYING or https://www.gnu.org/licenses/gpl-3.0.txt)  # noqa: E501

# cf. https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_general.html#creating-an-info-or-a-facts-module  # noqa: E501
from __future__ import (absolute_import, division, print_function)
__metaclass__ = type

DOCUMENTATION = r'''
---
module: hello_world

short_description: This is the hello_world module

version_added: "1.0.0"

description: A long description for the hello_world module

options:
    content:
        description: The content of the Hello_World-file
        required: true
        type: str

extends_documentation_fragment:
    - b1.ansible.hello_world

author:
    - tux (@yourGitHubHandle)
'''

EXAMPLES = r'''
### A SIMPLE INVOCATION
- name: Create a Hello_World-file
  b1.ansible.hello_world:
    content: hello world

### EXAMPLE FAILING THE MODULE
- name: Test the failure of the module
  b1.ansible.hello_world:
    content: FAILURE
'''

RETURN = r'''
### EXPLAIN POSSIBLE RETURN VALUES
content:
    description: The content which was written to the Hello_World-file
    type: str
    returned: always
    sample: 'goodbye'
'''

from ansible.module_utils.basic import AnsibleModule               # noqa: E402
from pathlib import Path                                           # noqa: E402
#  from ansible.module_utils.common.text.converters import to_native  # noqa: E402
#  from ansible.errors import AnsibleError                            # noqa: E402


def run_module():
    # definition of the yaml-keys the module provides
    module_args = dict(
        content=dict(type='str', required=True),
    )

    # create a result-dictionary
    # in case of an failure we can just return it
    result = dict(
        changed=False,
        failed=True,
        msg='Failed to instanciate the Hello_World-file with the specified content',  # noqa: E501
        content='None'
    )

    module = AnsibleModule(
        argument_spec=module_args,
        supports_check_mode=True
    )

    if module.check_mode:
        # we should check if the file would be created on the target nodes
        result['changed'] = True
        result['failed'] = False
        result['msg'] = "/home/tux/Hello_World would have been created"
        result['content'] = module.params['content']
        module.exit_json(**result)

    # provide the possibility to create a failure - only for demonstration purposes
    if module.params['content'] == 'FAILURE':
        result['content'] = module.params['content']
        result['msg'] = 'CATASTROPHIC FAILURE'
        module.fail_json(**result)

    HELLO_WORLD_FILE_PATH = Path("/home/tux/Hello_World")

    try:
        with open(HELLO_WORLD_FILE_PATH, mode="wt") as hello_world_file:
            hello_world_file.write(f"{module.params['content']} \n")

        # raise Exception("could not write to file")  # errorTesting
    except Exception as e:
        # AnsibleError can be used with ansible >=2.15
        # raise AnsibleError('Something happened, this was the original exception: %s' % to_native(e))  # noqa: E501
        result['msg'] = repr(e)
        module.fail_json(**result)

    result['changed'] = True
    result['failed'] = False
    # will be displayed with -v
    result['msg'] = "/home/tux/Hello_World has been created with the requested content"  # noqa: E501
    result['content'] = module.params['content']
    module.exit_json(**result)


def main():
    run_module()


if __name__ == '__main__':
    main()
    
```

4. configure the collection (check config-file for duplicated entries)

``` shell
echo -e "library = plugins\n" >> ~/code/b1/ansible/ansible.cfg
```

5. test the module

``` shell
cd ~/code/b1/ansible
```

``` shell
ansible -m hello_world -a 'content=test' localhost
ansible -m hello_world -a 'content=test' localhost -v

cat ~/Hello_World
```

6. args can be defined in a file

```shell
mkdir -p ~/code/b1/ansible/plugins/aux
cat <<'EOF' > ~/code/b1/ansible/plugins/aux/filter_args.json
{
    "ANSIBLE_MODULE_ARGS": {
        "content": "from file"
    }
}
EOF
```

execute

``` shell
python ~/code/b1/ansible/plugins/modules/hello_world.py ~/code/b1/ansible/plugins/aux/filter_args.json
```

7. use code in a playbook

`~/code/b1/ansible/playbooks/ModuleTest.yml`

``` yaml
- name: Test the new module
  hosts: localhost
  vars:
    content: 'Hello from the new module'
  tasks:
    - name: Run the new module
      hello_world:
        content: "{{ content }}"
        
```

8. run

``` shell
### run in CheckMode
ansible-playbook ./playbooks/ModuleTest.yml -vC
ansible-playbook ./playbooks/ModuleTest.yml -v
ansible-playbook ./playbooks/ModuleTest.yml -e '{"content": "overriden from CLI"}'   

ls ~/Hello_World

### Create a Fatal Error (only possible because we implemented it)
ansible-playbook ./playbooks/ModuleTest.yml -ve '{"content": "FAILURE"}'   
### more readable error messages
ANSIBLE_STDOUT_CALLBACK=yaml ansible-playbook ./playbooks/ModuleTest.yml -ve '{"content": "FAILURE"}'   

```
