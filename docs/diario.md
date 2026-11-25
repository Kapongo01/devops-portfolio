# Diário de formação

## Dia 1 — 19/11/2026 — Cultura DevOps e preparação
**Teoria:** CALMS, ciclo DevOps, métricas DORA.
**Prática:** WSL2 + Ubuntu 24.04, Git configurado, chave SSH ligada ao GitHub, repositório criado.

**O que aprendi:**
- DevOps = responsabilidade partilhada do código até produção.
- DORA: 2 métricas de velocidade (frequência de deploy, lead time) + 2 de estabilidade (taxa de falha, tempo de recuperação); melhoram juntas.
- `wsl` é um comando do Windows (PowerShell); dentro do Ubuntu não existe.
- `~/.ssh/known_hosts` guarda o fingerprint dos servidores para evitar ataques man-in-the-middle.

**Dúvidas / bloqueios:**
- Escrevi `wls` em vez de `wsl`.

**Próximo:** Dia 2 — sistema de ficheiros e permissões.

### Métricas DORA — ponto de partida (antes da formação)
Contexto: faço deploy em dois tipos de ambiente:
- **Servidores internos** cedidos pela DSI (aplicações internas, publicadas na rede interna);
- **Alojamento externo** (cPanel/FTP ou SSH) para sites públicos.

| Métrica | Como está hoje | Meta no fim do plano |
|---|---|---|
| **Frequência de deploy** | Irregular; agrupo várias alterações num deploy grande, feito à mão no servidor | Deploys pequenos e frequentes, disparados por merge no `main` |
| **Lead time for changes** | Dias; depende de ter tempo e acesso ao servidor (interno ou externo) | Minutos; pipeline faz build, testes e deploy |
| **Change failure rate** | Não medido; cada servidor é configurado à mão e pode diferir dos outros | Mesma imagem Docker em todos os ambientes; servidores configurados por Ansible (iguais e reproduzíveis) |
| **Tempo de recuperação** | Depende de descobrir a causa e de ter backup/versão anterior | Rollback para a imagem anterior; monitorização e alertas na rede interna |

**Desafios específicos dos servidores internos:**
- Não estão acessíveis a partir da internet → o GitHub Actions (cloud) não consegue fazer deploy direto; solução a estudar: *self-hosted runner* dentro da rede ou deploy por "pull".
- Dependo da DSI para criar/ceder servidores → automatizar a configuração (Ansible) reduz o tempo de preparar cada servidor novo.

**Ideia principal:** entregas pequenas e frequentes são *mais* seguras — menos risco por alteração e mais fácil de reverter.

## Dia 2 — 24–25/11/2026 — Sistema de ficheiros, utilizadores e permissões
**Teoria:** FHS, utilizadores/grupos, rwx e octal, setgid, sudo, umask.
**Prática:** utilizador `deploy` + grupo `www-data`; estrutura `/var/www/app` (tipo Laravel) com permissões de produção; 6 testes com `sudo -u`.

**O que aprendi:**
- Separar quem faz deploy (`deploy`, dono) de quem corre a app (`www-data`, só lê o código).
- Setgid (`2750`) faz os ficheiros novos herdarem o grupo da pasta.
- `.env` nasceu `644` (visível a todos) — corrigido para `640`.
- O erro do Laravel "storage/logs/laravel.log could not be opened" resolve-se com permissões certas, não com `777`.

**Descoberta importante:**
- Ficheiros criados pelo `deploy` nasceram `664` (umask `002` no Ubuntu) → o `www-data` conseguiu alterar código (`novo.php` passou de 0 para 12 bytes).
- Solução: repor permissões no fim de cada deploy (`find -exec chmod`). Vai entrar no script de deploy.

**Dúvidas / bloqueios:**
- Ao abrir `wsl` no PowerShell entrei na distro `docker-desktop` → resolvido com `wsl --set-default Ubuntu-24.04`.

**Próximo:** Dia 3 — processos, serviços (systemd) e pacotes.
