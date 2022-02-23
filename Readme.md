```yaml
---
- name: Install Elasticsearch **Установка Elasticsearch**   
  hosts: elasticsearch **Название хоста, с которым работаем**    
  handlers:   
    - name: restart Elasticsearch   
      become: true **Запрос повышенных прав**   
      service:   
        name: elasticsearch   
        state: restarted   
  tasks:   
    - name: "Download Elasticsearch's rpm" **Задание загрузки Elasticsearch**    
      get_url:   
        url: "https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-{{ elk_stack_version }}-x86_64.rpm" **ссылка на скачивание файла установки**    
        dest: "/tmp/elasticsearch-{{ elk_stack_version }}-x86_64.rpm" **куда скачиваем файл**    
      register: download_elastic **Переменная download_elastic**    
      until: download_elastic is succeeded   
    - name: Install Elasticsearch   
      become: true  **Запрос повышенных прав**     
      yum:    
        name: "/tmp/elasticsearch-{{ elk_stack_version }}-x86_64.rpm" **Ссылка на файл, который будем ставить**   
        state: present   
    - name: Configure Kibana **Конфигурирование Kibana**   
      become: true  **Запрос повышенных прав**   
      template:   
        src: elasticsearch.yml.j2     
        dest: /etc/elasticsearch/elasticsearch.yml **Место назначения файла**   
      notify: restart Elasticsearch   
   
- name: Install Kibana   
  hosts: kibana **Название хоста, с которым работаем**    
  handlers:   
    - name: restart Kibana   
      become: true  **Запрос повышенных прав**    
      service:   
        name: kibana   
        state: restarted   
  tasks:   
    - name: "Download Kibana's rpm"   
      get_url:    
        url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ elk_stack_version }}-x86_64.rpm" **ссылка на скачивание файла установки**   
        dest: "/tmp/kibana-{{ elk_stack_version }}-x86_64.rpm" **куда скачиваем файл**   
      register: download_kibana **Переменная download_kibana**    
      until: download_kibana is succeeded **сообщение об успешной установке**   
    - name: Install Kibana   
      become: true  **Запрос повышенных прав**   
      yum:   
        name: "/tmp/kibana-{{ elk_stack_version }}-x86_64.rpm"   
        state: present   
    - name: Configure Kibana   
      become: true  **Запрос повышенных прав**   
      template:   
        src: kibana.yml.j2   
        dest: /etc/kibana/kibana.yml   
      notify: restart Kibana    
   
- name: Install filebeat   
  hosts: app **Название хоста, с которым работаем**    
  handlers:   
    - name: restart filebeat   
      become: true  **Запрос повышенных прав**   
      systemd:   
        name: filebeat    
        state: restarted   
        enabled: true    
  tasks:   
    - name: "Download filebeat's rpm"    
      get_url:   
        url: "https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-{{ elk_stack_version }}-x86_64.rpm" **ссылка на скачивание файла установки**   
        dest: "/tmp/filebeat-{{ elk_stack_version }}-x86_64.rpm" **куда скачиваем файл**   
      register: download_filebeat   
      until: download_filebeat is succeeded    
    - name: Install filebeat   
      become: true  **Запрос повышенных прав**    
      yum:   
        name: "/tmp/filebeat-{{ elk_stack_version }}-x86_64.rpm"    
        state: present     
      notify: restart filebeat   
    - name: Configure filebeat   
      become: true  **Запрос повышенных прав**    
      template:   
        src: filebeat.yml.j2   
        dest: /etc/filebeat/filebeat.yml   
      notify: restart filebeat   
    - name: Set filebeat systemwork   
      become: true  **Запрос повышенных прав**    
      command:   
        cmd: filebeat modules enable system **Запуск filebeat**    
        chdir: /usr/share/filebeat/bin   
      register: filebeat_modules **Переменная filebeat_modules**    
      changed_when: filebeat_modules.stdout != 'Module system is already enabled' **Проверка неравенства вывода переменной**    
    - name: Load Kibana dashboard   
      become: true  **Запрос повышенных прав**    
      command:   
        cmd: filebeat setup   
        chdir: /usr/share/filebeat/bin   
      register: filebeat_setup    
      changed_when: false    
      until: filebeat_setup is succeeded    
```
