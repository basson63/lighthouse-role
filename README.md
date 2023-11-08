lighthouse-role
=========

This role install [lighthouse](https://github.com/VKCOM/lighthouse)

Role Variables
--------------

| Variable name | Default | Description |
|-----------------------|----------|-------------------------|
| lighthouse_dir  | "/usr/share/nginx/html/lighthouse" | Дериктория в которую будет установлен lighthouse |
| lighthouse_url  | "https://github.com/VKCOM/lighthouse.git" | Ссылка, откуда для скачивания дистрибутива lighthouse |
| lighthouse_port | "80" | Порт, на котором будет веб интерфейс отвечать |

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - lighthouse-rile

License
-------

MIT

Author Information
------------------

Yaroslav Komlev
