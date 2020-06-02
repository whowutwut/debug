# Ansible Collection - victor.testing

Documentation for the collection.



## Debugging Notes

The try/except blocks here look like this: 

```python
try:
   print("attempt ansible collections...")
   from ansible_collections.victor.testing.plugins.module_utils.my_utils import helloWorld
except:
   print("exception from collections...")
   try:
      print("attempt ansible.module_utils...")
      from ansible.module_utils.my_utils import helloWorld
   except ImportError:
      print("attempt regular import...")
      from my_utils import helloWorld
```

Running python works fine.. we see it falls through correctly...

```bash
# ./run.python
+ PYTHONPATH=/root/ansible/lib:./library:./module_utils
+ python -m my_test ./args.json
attempt ansible collections...
exception from collections...
attempt ansible.module_utils...
attempt regular import...
Hello World Return Message

{"invocation": {"module_args": {"new": true, "name": "hello"}}, "message": "Hello World Return Message", "changed": true, "original_message": "hello"}
#
```

Running through ansible, has some problems....

```bash
# ./run.ansible
+ ansible-playbook test.yml
[WARNING]: You are running the development version of Ansible. You should only run
Ansible from "devel" if you are modifying the Ansible engine, or trying out features
under development. This is a rapidly changing source of code and can become unstable at
any point.
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the
implicit localhost does not match 'all'

PLAY [localhost] ************************************************************************

TASK [Gathering Facts] ******************************************************************
ok: [localhost]

TASK [Test the my_test module.] *********************************************************
fatal: [localhost]: FAILED! => {"msg": "Could not find imported module support code for my_test.  Looked for either helloWorld.py or my_utils.py"}

PLAY RECAP ******************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
```


It seems to be caused by the `plugins` in the path: `from ansible_collections.victor.testing.plugins.module_utils.my_utils import helloWorld` .. If I remove it, it works:

```bash
# ./run.ansible
+ ansible-playbook test.yml
^[[A[WARNING]: You are running the development version of Ansible. You should only run
Ansible from "devel" if you are modifying the Ansible engine, or trying out features
under development. This is a rapidly changing source of code and can become unstable at
any point.
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the
implicit localhost does not match 'all'

PLAY [localhost] ************************************************************************

TASK [Gathering Facts] ******************************************************************
ok: [localhost]

TASK [Test the my_test module.] *********************************************************
changed: [localhost]

TASK [Debug the my_test module's output] ************************************************
ok: [localhost] => {
    "msg": {
        "changed": true,
        "failed": false,
        "message": "Hello World Return Message",
        "original_message": "hello"
    }
}

PLAY RECAP ******************************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

But also, copying this collection to `~/.ansible/collections/ansible_collections/` and `plugins` added back to the path works....


