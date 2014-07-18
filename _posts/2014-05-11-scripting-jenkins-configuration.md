---
layout: page
---

Out of the box Jenkins is pretty basic, some would say nothing more than a glorified cron. But that's awesome, because it's so easily customizable and fully extendable via its plugin system it doesn't need to be any more.

When setting up my Jenkins server with Ansible I wanted to provide a default set of jobs that had all the basic functionality to build various projects, create packages, etc... Everyone using this playbook to provision their VM could add extra jobs as needed, but the standard shared jobs would be ready to go. For these jobs to do anything useful though would obviously require some plugins (eg. Git, Publish over SSH and Build Name) - so how to install them?

Jenkins comes with a CLI tool you can download from each installation, and has a bunch of commands, here's a snippet of the help output...

```
[vagrant@vagrant-centos65]/var/lib/jenkins% java -jar ./cli.jar -s http://127.0.0.1:8080
  build
    Builds a job, and optionally waits until its completion.
  cancel-quiet-down
    Cancel the effect of the "quiet-down" command.
  clear-queue
    Clears the build queue.
  connect-node
    Reconnect to a node.
  console
    Retrieves console output of a build.
  copy-job
    Copies a job.
  create-job
    Creates a new job by reading stdin as a configuration XML file.
  create-node
    Creates a new node by reading stdin as a XML configuration.
  etc...
```

Thankfully there's one to install plugins, which I can easily use from an Ansible command. But running this on a clean Jenkins install you'll get an error saying the plugin you're trying to install isn't found. In fact, no plugins will be found!

The reason for this is that Jenkins fetches all the plugin information from its update center, and its that data it uses to display information about what is available. So before it's done this (ie. when you've just provisioned the instance) it won't have any of this information. I found a neat trick in a Stackoverflow (where else?) thread about how to manually do a sync of this information, which I use in my playbook just after installing Jenkins and ensuring it's running...

```
- name: Ensure jenkins Running
  service: name=jenkins state=started

- name: Ensure Jenkins Is Available
  wait_for: port=8080 delay=5

- name: Ensure Jenkins CLI Present
  copy: src=jenkins-cli.jar dest=/var/lib/jenkins/cli.jar
        owner=jenkins group=jenkins mode=755

- name: Ensure Jenkins Update Directory Exists
  file: path=/var/lib/jenkins/updates state=directory
        owner=jenkins group=jenkins

- name: Ensure Jenkins Update Center Synced
  shell: wget http://updates.jenkins-ci.org/update-center.json -qO- | sed '1d;$d' > /var/lib/jenkins/updates/default.json

- name: Ensure Jenkins Update Center Data Permissions
  file: path=/var/lib/jenkins/updates/default.json state=file
        owner=jenkins group=jenkins mode=644
```

Jenkins will then have all the information it needs about the latest plugins, and you can install away...

```
- name: Ensure Jenkins Plugins Installed
  shell: java -jar /var/lib/jenkins/cli.jar -s http://127.0.0.1:8080 install-plugin git xunit publish-over-ssh ws-cleanup checkstyle ansicolor rebuild jslint build-name-setter
  notify:
      - restart jenkins www
```

You'll need to do a restart of Jenkins for these to be available, I'm using an Ansible handler.

Finally to create the default jobs I keep the XML configuration for each versioned in my Ansible repository, then copy these in and use the CLI tool to install them.

```
- name: Install Job Configuration
  copy: src={{ item }}.xml dest=/tmp/{{ item }}.xml
  with_items:
      - some-job
      - another-one

- name: Load Default Job Configuration
  shell: java -jar /var/lib/jenkins/cli.jar -s http://127.0.0.1:8080 create-job {{ item }} < /tmp/{{ item }}.xml || true
  with_items:
      - some-job
      - another-one
```

Any or all of this might be complete madness of course, and there could be a much better way - so please let me know if I can make any simplifications.  Otherwise, hope the information helps someone.

