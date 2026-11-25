# Cábula Linux — Kapongo José

## Hierarquia de pastas (FHS)
| Pasta | Para que serve |
|---|---|
| `/etc` | Configurações do sistema e serviços (`/etc/nginx`, `/etc/passwd`) |
| `/var` | Dados variáveis: logs (`/var/log`), sites (`/var/www`) |
| `/home` | Pastas pessoais dos utilizadores |
| `/root` | Pasta pessoal do root |
| `/usr/bin` | Programas do sistema |
| `/opt` | Software instalado fora do `apt` |
| `/tmp` | Temporários (apagados no reboot) |

## Utilizadores e grupos
```bash
whoami                         # utilizador atual
id <user>                      # UID, GID e grupos
sudo adduser <user>            # cria utilizador + /home + grupo com o mesmo nome
sudo usermod -aG <grupo> <user>  # junta a um grupo (SEM -a substitui todos os grupos!)
grep <user> /etc/passwd        # linha do utilizador
```
- UID 0 = root · UID < 1000 = utilizadores de serviço (ex.: `www-data` = 33) · UID ≥ 1000 = pessoas
- Ficheiros: `/etc/passwd` (utilizadores), `/etc/shadow` (passwords), `/etc/group` (grupos)

## Permissões
```
-rw-r----- 1 deploy www-data  index.php
 └┬┘└┬┘└┬┘
  u  g  o     u=dono  g=grupo  o=outros
```
| Letra | Valor | Ficheiro | Pasta |
|---|---|---|---|
| r | 4 | ler | listar (`ls`) |
| w | 2 | alterar | criar/apagar ficheiros dentro |
| x | 1 | executar | entrar (`cd`) |

| Octal | Uso |
|---|---|
| `755` / `644` | Pastas / ficheiros públicos (padrão) |
| `2750` / `640` | Pastas / ficheiros de app em produção |
| `600` | Segredos (`.env`, chaves SSH) |

```bash
sudo chown -R deploy:www-data /caminho   # muda dono:grupo (recursivo)
sudo chmod 640 ficheiro                  # permissões em octal
sudo chmod -R g+w pasta                  # simbólico: dá escrita ao grupo
sudo find /caminho -type d -exec chmod 2750 {} \;   # só pastas
sudo find /caminho -type f -exec chmod 640 {} \;    # só ficheiros
ls -la / ls -ld pasta                    # ver permissões (-d = a própria pasta)
```
- **Setgid em pasta** (`2` à frente, aparece como `s`): ficheiros novos herdam o **grupo da pasta**.
- Nunca `chmod -R 777` — dá escrita a qualquer processo do servidor.

## sudo
```bash
sudo <comando>                 # corre como root (fica em log)
sudo -u <user> <comando>       # corre como outro utilizador (ótimo para testar permissões)
echo "texto" | sudo tee ficheiro > /dev/null   # escrever ficheiro com sudo
```
- `sudo echo x > ficheiro` **não funciona**: o `>` é feito pela minha shell, sem privilégios.
- Princípio do menor privilégio: cada processo só com as permissões de que precisa.

## umask
```bash
umask                          # ver a máscara atual
```
- `022` → ficheiros nascem `644`, pastas `755`
- `002` → ficheiros nascem `664` (Ubuntu usa isto quando o grupo primário tem o nome do utilizador)

## Operadores úteis
```bash
A && B     # B só corre se A funcionar
A || B     # B só corre se A falhar
cmd 2>/dev/null   # esconde mensagens de erro
```

## Receita: permissões de uma app Laravel em produção
```bash
sudo chown -R deploy:www-data /var/www/app
sudo find /var/www/app -type d -exec chmod 2750 {} \;
sudo find /var/www/app -type f -exec chmod 640 {} \;
sudo chmod -R g+w /var/www/app/storage /var/www/app/bootstrap/cache
```
- `deploy` = dono do código · `www-data` (PHP/Nginx) = só lê o código, só escreve em `storage` e `bootstrap/cache`.
- **Repor estas permissões no fim de cada deploy** — ficheiros novos nascem com o umask de quem os cria.
