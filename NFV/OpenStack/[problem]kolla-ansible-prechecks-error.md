# Kolla-Ansible部署前执行`kolla-anisible prechecks`时rabbitmq报错`Hostname has to resolve to IP address of api_interface`

## 问题描述

执行
```
(venv) # kolla-ansible prechecks
```
报错
```
failed: [localhost] (item=[{'cmd': ['getent', 'ahostsv4', 'controller'], 'stdout': '10.112.xxx.xxx   STREAM controller\n10.112.xxx.xxx   DGRAM  \n10.112.xxx.xxx   RAW    \n172.17.0.1      STREAM \n172.17.0.1      DGRAM  \n172.17.0.1      RAW    ', 'stderr': '', 'rc': 0, 'start': '2022-10-07 21:45:20.537806', 'end': '2022-10-07 21:45:20.542234', 'delta': '0:00:00.004428', 'changed': False, 'invocation': {'module_args': {'_raw_params': 'getent ahostsv4 controller', 'warn': True, '_uses_shell': False, 'stdin_add_newline': True, 'strip_empty_ends': True, 'argv': None, 'chdir': None, 'executable': None, 'creates': None, 'removes': None, 'stdin': None}}, 'stderr_lines': [], 'failed': False, 'item': 'localhost', 'ansible_loop_var': 'item'}, '172.17.0.1      STREAM ']) => {"ansible_loop_var": "item", "changed": false, "item": [{"ansible_loop_var": "item", "changed": false, "cmd": ["getent", "ahostsv4", "controller"], "delta": "0:00:00.004428", "end": "2022-10-07 21:45:20.542234", "failed": false, "invocation": {"module_args": {"_raw_params": "getent ahostsv4 controller", "_uses_shell": false, "argv": null, "chdir": null, "creates": null, "executable": null, "removes": null, "stdin": null, "stdin_add_newline": true, "strip_empty_ends": true, "warn": true}}, "item": "localhost", "rc": 0, "start": "2022-10-07 21:45:20.537806", "stderr": "", "stderr_lines": [], "stdout": "10.112.xxx.xxx   STREAM controller\n10.112.xxx.xxx   DGRAM  \n10.112.xxx.xxx   RAW    \n172.17.0.1      STREAM \n172.17.0.1      DGRAM  \n172.17.0.1      RAW    "}, "172.17.0.1      STREAM "], "msg": "Hostname has to resolve uniquely to the IP address of api_interface"}
```

## 解决方法

修改host文件：
```
<nic的ip>   localhost
<nic的ip>   <hostname>
```
即
```
10.112.xxx.xxx  localhost
10.112.xxx.xxx  controller