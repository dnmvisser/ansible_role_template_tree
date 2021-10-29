# ansible_role_template_tree

Deploys an entire directory of templates.

Benefits over `template` + `filetree` or `fileglob` lookup:

- Supports deletion of remote content that is not available locally (i.e.
  _syncing_).
- Configurable set of file name patterns to template, so that the rest of the
  files are not templated but copied as-is. Useful for mixed content, for
  example a web server directory containing both scripts and images.
- Configurable file validation per file name pattern. Useful to validate
  PHP, python, XML, JSON, YAML, etc.
- Files that are not templated but copied as-is can also be validated. This can
  be useful for example to verify the integrity of images, zip files, etc.
- Can strip part(s) of the file name. Defaults to stripping `.j2` extensions.


# Example 1

- Sync a web server directory
- Delete all remote content that is not available locally - except the `.cache`
  directory
- Template all `.php`, `.html`, and `.css` files
- Copy all other files as-is
- Validate all `.php` files

```yaml
- hosts: web
  become: yes

  tasks:
    - include_role:
        name: ansible_role_template_tree
      vars:
        tt_src_dir: /opt/html
        tt_dest_dir: /var/www/html
        tt_template_pattern: '\.(php|html|css)$'
        tt_delete: yes
        tt_delete_exclude_pattern: '^.cache'
        tt_validate:
          - desc: PHP files
            pattern: '\.php$'
            validate: 'php -l %s'
```

# Example 2

Same as above, but also verify the integrity of any images using the `identify` command (from the ImageMagick package):

```yaml
- hosts: web
  become: yes

  tasks:
    - include_role:
        name: ansible_role_template_tree
      vars:
        tt_src_dir: /opt/html
        tt_dest_dir: /var/www/html
        tt_template_pattern: '\.(php|html|css)$'
        tt_delete: yes
        tt_delete_exclude_pattern: '^.cache'
        tt_validate:
          - desc: PHP files
            pattern: '\.php$'
            validate: 'php -l %s'
          - desc: Image files
            pattern: '\.(?i)(jpe?g|png|gif|bmp)$'
            validate: 'identify %s'
```

# Example 3

- Sync a Tomcat configuration directory
- Template all files
- Validate the XML files using `xmllint`

Including roles [does not support a 'changed'
feature](https://github.com/ansible/ansible/issues/26537). Therefore, an extra task
is needed to implement this logic.


```yaml
- hosts: web
  become: yes

  handlers:
    - service:
        name: tomcat9
        state: restarted
      listen: restart_tomcat

  tasks:
    - include_role:
        name: ansible_role_template_tree
      vars:
        tt_src_dir: /opt/config/tomcat
        tt_dest_dir: /etc/tomcat9
        tt_owner: root
        tt_group: tomcat
        tt_delete: yes
        tt_validate:
          - desc: XML files
            pattern: '\.xml$'
            validate: 'xmllint %s'

    - debug:
        msg: Restarting services if needed
      changed_when: tt_changed|bool
      notify: restart_tomcat
```
