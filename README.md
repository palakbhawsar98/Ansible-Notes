# Ansible
Playbooks are the files where the Ansible code is written. Playbooks are written in YAML format. YAML means "Yet Another Markup Language,"

#Vars : Vars defines the variables which you can use in your playbook. Its usage is similar to the variables in any programming language.
Example:vars:
        car_model: 'BMW'
        country_name: 'USA'
        title: 'Systems Engineer'
        
#Conditions: Conditions in playbook are same as 'if/else' conditions in programming   
Syntax: tasks:
         - name: Install httpd service
           yum: 
                name: httpd
                state: present
           when: ansible_facts['distribution'] == 'Ubuntu'
           
 #Loops: Loops are used to iterate the condition in playbook for diffrent servers
 Example: - name: Add several users
            ansible.builtin.user:
                name: "{{ item }}"
                state: present
                groups: "wheel"
            loop:
              - testuser1
              - testuser2
            
