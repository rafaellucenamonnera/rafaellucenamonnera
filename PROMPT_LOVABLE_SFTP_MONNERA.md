# Prompt para aplicar no Lovable (adequações SFTP Monnera)

Você é um assistente de desenvolvimento atuando sobre um código **já existente**. Faça somente mudanças necessárias para viabilizar o envio via SFTP para a Monnera, **sem alterações desnecessárias ou redundantes**.

## Objetivo
Implementar de ponta a ponta (frontend + backend) o fluxo de:
1. gerar/validar arquivo no padrão Monnera,
2. enviar via SFTP,
3. acompanhar status de processamento,
4. permitir exclusão e reenvio pelo frontend (sem depender de banco de dados/manual técnico).

---

## Regras obrigatórias de formato do arquivo (Monnera)
Use automaticamente os dados já disponíveis no sistema para preencher os campos exigidos:

- Gerar **1 arquivo por empresa**.
- Arquivo **sem cabeçalho** (somente dados).
- Separador de campos: `|`.
- Se campo alfanumérico tiver `|` no conteúdo, substituir por `char(124)` (ou remover, conforme configuração), antes de exportar.
- Booleanos: converter para `True`/`False`.
- Data/hora: `yyyyMMddHHmmss` (24h).
- Data: `yyyyMMdd`.
- Nome do arquivo: `{cnpj}_{yyyyMMddHHmmss}.csv`.
  - Exemplo: `12345678901234_20231231120000.csv`.

Criar validação automática antes do envio e bloquear envio quando o arquivo estiver fora do padrão, exibindo erro claro no frontend.

---

## SFTP Monnera (backend)
Implementar cliente SFTP no backend com:

- Host: `sftp.monnera.com.br`
- Porta: `2222`
- Autenticação por chave privada `.pem`
- Usuário: CNPJ da contratante
- Diretório remoto padrão de upload: `arquivos` (configurável)

### Configuração segura e automática
- Criar módulo de configuração com variáveis de ambiente, com fallback seguro.
- Não expor secrets no frontend.
- Criar estrutura de armazenamento de chaves por cliente:
  - Ex.: `/secure-keys/{cliente_nome}/monnera/private_key.pem`
- Criar rota/backend service para resolução automática da chave correta por cliente no momento do envio.
- Registrar logs técnicos (sem expor chave/sigilosos) com correlation-id por envio.

### Campos que dependem do time Monnera
Quando faltar informação externa, criar campos/configurações com instrução explícita no label/placeholder:
- `Solicite ao time Monnera a informação: usuário SFTP (CNPJ)`
- `Solicite ao time Monnera a informação: chave privada .pem válida`
- `Solicite ao time Monnera a informação: diretório remoto final (caso diferente de /arquivos)`
- `Solicite ao time Monnera a informação: whitelist de IP (se exigido)`

---

## Frontend (obrigatório)
Criar/ajustar telas para usuários não técnicos:

1. **Tela de envio Monnera**
   - Seleção da empresa/cliente.
   - Geração do arquivo no padrão e pré-validação.
   - Botão `Enviar via SFTP`.
   - Exibição de status: `Pendente`, `Enviado`, `Falha`, `Processado`, `Não processado`.
   - Exibir mensagens detalhadas de erro (ex.: falha de autenticação, permissão, timeout, layout inválido).

2. **Ações de manutenção via frontend** (fundamental)
   - Se não existir, criar botão: **`Excluir arquivo enviado`**.
   - Criar também: **`Reenviar arquivo`** e **`Baixar arquivo gerado`**.
   - Confirmar exclusão com modal e trilha de auditoria (quem, quando, motivo).

3. **Histórico de envios**
   - Lista por cliente com data/hora, nome do arquivo, hash/checksum, status e tentativas.
   - Filtros por período/status/empresa.

---

## Estrutura por cliente (pastas/repositórios)
Se necessário para aderir ao padrão de operação, criar estrutura por cliente usando nome do cliente no repositório:
- `clientes/{cliente_nome}/outbox/`
- `clientes/{cliente_nome}/sent/`
- `clientes/{cliente_nome}/failed/`
- `clientes/{cliente_nome}/logs/`

Movimentar arquivo automaticamente entre pastas conforme resultado do envio/processamento.

---

## WinSCP (configuração assistida)
Adicionar no frontend/backoffice uma seção “Configuração WinSCP” com instruções prontas para cópia:

- Hostname: `sftp.monnera.com.br`
- Port Number: `2222`
- User name: `{usuario_cnpj}`
- Advanced > Authentication > Private key file: `{caminho_da_chave_pem}`
- Remote directory: `arquivos`

Gerar botão `Copiar configurações WinSCP` e botão `Testar conexão SFTP`.

---

## Regras técnicas de implementação
- Reaproveitar código existente sempre que possível.
- Não renomear entidades sem necessidade.
- Não quebrar APIs atuais.
- Incluir migração de dados **somente se necessário**.
- Cobrir com testes unitários/integrados:
  - formatação do arquivo,
  - sanitização de `|`,
  - nome do arquivo,
  - autenticação com chave,
  - upload SFTP,
  - exclusão e reenvio.

---

## Critérios de aceite (checklist)
- [ ] Arquivo exportado 100% no padrão Monnera.
- [ ] Envio SFTP funcional com chave `.pem`.
- [ ] Mensagem de sucesso só aparece com confirmação real de upload.
- [ ] Usuário consegue excluir/reenviar via frontend.
- [ ] Campos faltantes marcados com “Solicite ao time Monnera a informação: ...”.
- [ ] Configuração WinSCP disponível e copiável.
- [ ] Logs e auditoria habilitados.

Implemente agora com o menor conjunto de mudanças necessário, preservando a arquitetura atual.
