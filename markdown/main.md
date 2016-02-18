Using the
### hastexo XBlock
with
### OpenStack

(Press space to proceed, or use your arrow keys.)


### XML

This is how a course author invokes the hastexo XBlock XML in the courseware:

```xml
<vertical url_name="lab_introduction">
  <hastexo
    url_name="lab_introduction"
    stack_template_path="hot_lab.yaml"
    instructions_path="markdown_lab.md"
    stack_user_name="training"
    os_auth_url="https://ops.elastx.net:5000/v2.0"
    os_tenant_name="example.com"
    os_username="demo@example.com"
    os_password="foobarfoobarfoofoo" />
</vertical>
```
<!-- .element: class="fragment" -->


### Authentication

The XBlock supports standard OpenStack authentication variables, including
Keystone v3:

* os\_auth\_url<!-- .element: class="fragment" -->
* os\_tenant\_name<!-- .element: class="fragment" -->
* os\_username<!-- .element: class="fragment" -->
* os\_password<!-- .element: class="fragment" -->



### Heat

The uploaded Heat template (`stack_template_path`) defines the lab environment
itself.  It is a regular Heat Template, but it needs to meet the following
criteria...


### 1. Public IP

The template must assign a public IP address to one node in the environment,
and provide it as a template output named `public_ip`:

```python
resources:
  deploy_floating_ip:
    type: OS::Neutron::FloatingIP
    properties:
      floating_network_id: { get_param: public_net_id }
      port_id: { get_resource: deploy_management_port }
      fixed_ip_address: 192.168.122.100

output:
  public_ip:
    description: Floating IP address of deploy in public network
    value: { get_attr: [ deploy_floating_ip, floating_ip_address ] }
```
<!-- .element: class="fragment" -->


### 2. Training Key

The template must also create a private SSH key identified by the stack's own
name, and provide it as a `private_key` output:

```python
resources:
  training_key:
    type: OS::Nova::KeyPair
    properties:
      name: { get_param: 'OS::stack_name' }
      save_private_key: true
  
outputs:
  private_key:
    description: Training private key
    value: { get_attr: [ training_key, private_key ] }
```
<!-- .element: class="fragment" -->


### 3. Training Key Assignment

The training key must be usable to access not only the public node, but all
others.  To do so, a specific system user can be defined using the Heat
CloudConfig resource, and the private and public keys deployed as such:

```python
resources:
  cloud_config:
    type: OS::Heat::CloudConfig
    properties:
      cloud_config:
        users:
          - name: training
            gecos: Training User
            groups: users,adm
            ssh-authorized-keys:
              - { get_attr: [ training_key, public_key ] }
        write_files:
          - path: /home/training/.ssh/id_rsa
            permissions: '0600'
            content: { get_attr: [ training_key, private_key ] }
```
<!-- .element: class="fragment" -->


### Lab Instructions

Lab instructions are written in markdown, one file per lab (also uploaded to
the course content store), and will be rendered as HTML immediately above the
terminal window.


### Grading

Labs can be graded by defining tests in any available scripting language .  A
return value of `0` means the student was successful.  Anything else means
failure:

```xml
<hastexo>
  <test>
    #!/bin/bash -e
    for i in daisy eric frank; do
      ssh $i dpkg -l ceph
    done
  </test>
  <test>
    #!/bin/bash -e
    # Check if directory is present.
    ls -l /etc/ceph
  </test>
</hastexo>
```
<!-- .element: class="fragment" -->


## Screenshots

This is what the student sees, in practice:


<!-- .slide: data-background-image="images/screenshot1.png" data-background-size="contain" -->


<!-- .slide: data-background-image="images/screenshot2.png" data-background-size="contain" -->


<!-- .slide: data-background-image="images/screenshot3.png" data-background-size="contain" -->


<!-- .slide: data-background-image="images/screenshot4.png" data-background-size="contain" -->


<!-- .slide: data-background-image="images/screenshot5.png" data-background-size="contain" -->


<!-- .slide: data-background-image="images/by-sa.svg" data-background-size="contain" -->
http://arbrandes.github.io/hastexo-xblock-openstack-integration

https://github.com/arbrandes/hastexo-xblock-openstack-integration

This presentation, like most hastexo slide decks, is under the CC-BY-SA 3.0
license. So please feel free to peruse these slides as a basis for your own
presentations, as you see fit.
