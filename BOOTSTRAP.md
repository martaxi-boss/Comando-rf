# BOOTSTRAP — Comando RF

> Estado verificado em 14/11/2026. Este documento regista decisões e limites reais; não certifica hardware que ainda não foi montado/testado.

## 1. Identidade e fonte de verdade

- Projeto do proprietário: https://github.com/martaxi-boss/Comando-rf
- Firmware base: Bruce Firmware, organização **BruceDevices**, repositório https://github.com/BruceDevices/firmware
- Release de referência: **1.17**, publicada em **12/11/2026**: https://github.com/BruceDevices/firmware/releases/tag/1.17
- Commit original imobilizado: `2b6ba4d490bfcf94ce2574c6154537dbd5838c3d`.
- Proveniência técnica: [COMANDO_RF_UPSTREAM.md](COMANDO_RF_UPSTREAM.md). Esta é a referência para apurar a versão incorporada no projeto.
- Código-fonte Bruce 1.17 importado; automatizações originais não foram ativadas e estão em `upstream-ci-original/`.
- Licença original: **AGPL-3.0-or-later**; conservar avisos, licença e obrigações de disponibilização do código alterado.

## 2. Objetivo e limites

Montar um aparelho portátil ESP32-C5 com ecrã tátil, transceptor Sub-GHz CC1101 e armazenamento opcional, preservando o Bruce. Permitir a análise e utilização de comandos **próprios ou expressamente autorizados**, sem pressupor que uma captura de sinal serve como credencial de emparelhamento.

**Não alterar o código RF, não criar ainda o botão «Abrir portão» e não afirmar que há abertura de portão ou emparelhamento certificados.** A funcionalidade só poderá ser proposta depois de se confirmar o protocolo específico, o processo legítimo de inscrição na central, a persistência correta dos contadores e o comportamento em teste físico autorizado. Não contornar encriptação, não extrair chaves de terceiros e não transmitir onde não há autorização.

## 3. Material definido para o primeiro protótipo

| Quantidade | Componente | Especificação crítica |
| --- | --- | --- |
| 1 | ESP32-C5 DevKitC-1-N8R8, ou equivalente com GPIO expostos | ESP32-C5, flash 8 MB; confirmar pinout acessível |
| 1 | TFT tátil SPI 2,8 pol. 240 × 320 | Controlador **ILI9341**, tátil **XPT2046**; não trocar por ST7789 sem mudar firmware |
| 1 | CC1101 Sub-GHz | Variante ajustada à banda **433 MHz**, lógica e alimentação **3,3 V**, de preferência conector SMA |
| 1 | Antena 433 MHz | Conector e banda adequados ao CC1101 |
| 1 | Breadboard/perfboard | Para prototipagem, antes de fixação permanente |
| 1 conjunto | Jumpers e cabos | Pinos compatíveis, curtos e bem isolados |
| 1 | Cabo USB de dados | Compatível com a placa ESP32-C5, não apenas carregamento |
| opcional | microSD + leitor/slot SPI | Se o ecrã incluir slot compatível, usar CS GPIO10; caso contrário montar slot separado |
| opcional | Caixa e power bank USB | Usar alimentação USB certificada; não ligar célula Li-ion diretamente ao ESP32-C5 |

**Antes de comprar:** verificar a referência exata do TFT (ILI9341 + XPT2046), os pinos disponíveis na placa C5, e a **frequência real da instalação autorizada**. O ESP32-C5 suporta Wi-Fi 2,4/5 GHz, mas o CC1101 é rádio Sub-GHz; uma frequência não substitui a outra.

## 4. Montagem, pinos e segurança

Consultar [docs/ESQUEMA_MONTAGEM.md](docs/ESQUEMA_MONTAGEM.md) e o [pinout original Bruce](boards/ESP32-C5-tft/connections.md).

- SPI partilhado: GPIO **6=SCK**, **7=MOSI**, **2=MISO**.
- TFT ILI9341: **CS=23**, **DC=24**, **BL=25**, reset segundo as especificações da placa.
- Touch XPT2046: **CS=1**; CLK/DIN/DO partilham o SPI.
- CC1101: **CS=9**, **GDO0=8**.
- microSD SPI opcional: **CS=10**.
- Alimentações e GND comuns: **3,3 V lógico**; **nunca** aplicar 5 V ao CC1101 nem GPIO do ESP32-C5.
- CC1101 e NRF24, se ambos forem usados no futuro, partilham pinos CS e GDO0/CE nesta configuração; exigir seleção isolada, não ligar simultaneamente sem comutação projetada.
- No modo tátil ILI9341/XPT2046 não presumir os três botões físicos (a opção com botões do Bruce é alternativa de ecrã).

Montagem física **ainda não testada**. Fazer primeiro ensaio de ecrã e navegação; depois verificar CC1101 sem emissões, só depois ensaios autorizados.

## 5. Instalação de firmware — disponibilidade real

**Disponível no autor, não como release compilada própria deste repositório:** [Bruce-esp32-c5-tft.bin para ILI9341 + XPT2046](https://github.com/BruceDevices/firmware/releases/download/1.17/Bruce-esp32-c5-tft.bin).

- Versão: `1.17`; ficheiro: `Bruce-esp32-c5-tft.bin`; tamanho listado pelo GitHub: **4.556.352 bytes**.
- SHA-256 anunciado no release oficial: `fedd5a79231f85c217627f02d86afa4c42d47349b406293bc35fdbf4b8d4d0f8`.
- Instruções oficiais: https://bruce.computer/flasher e https://github.com/BruceDevices/firmware#how-to-install.
- Para gravação manual, depois de verificar a porta correta e guardar dados necessários:
  `python -m esptool --chip esp32c5 --port COMx write_flash 0x00000 Bruce-esp32-c5-tft.bin`
  (substituir `COMx` pela porta real do Windows; no Linux usar o dispositivo correspondente). Validar SHA-256 do ficheiro descarregado antes de gravar.
- **Não** escolher `Bruce-esp32-c5.bin` (sem TFT) nem variante ST7789 para este ecrã.
- Não há ainda prova de compilação independente, gravação real em placa, testes de toque/RF ou instalação OTA do fork. Não anunciar instalação concluída antes desses testes.

## 6. O que Bruce 1.17 já contém e o que falta provar

| Área | Estado |
| --- | --- |
| RF Scan/Copy, gravação de sinais `.sub`, Custom SubGhz, RF Emulate | Código existente; hardware não verificado |
| Suporte CC1101 e configuração de frequência | Código existente; verificar módulo/antena |
| KeeLoq compatível, geração de tramas e incremento do contador | Caminhos de código existentes; compatibilidade depende do recetor |
| Persistência de contador em RF Emulate | Implementada em caminho específico, sem garantia de transação atómica |
| Persistência universal em Scan/Copy e ProtoPirate | **Não garantida** |
| Emparelhamento autorizado com central de portão | **Não validado** |
| Confirmação física de abertura pelo recetor | **Não implementada** |
| Botão dedicado «Abrir portão» | **Não criado**; aguardar validação |

**Portas de aceitação futuras:** verificar protocolo, frequência e autorização; método oficial de emparelhamento; contador persistido com integridade e recuperação após falha; não usar transmissão como prova de abertura; teste físico autorizado, logs e regressões.

## 7. Acompanhamento de novas versões do autor

Workflow: [`.github/workflows/watch-bruce-releases.yml`](.github/workflows/watch-bruce-releases.yml).

- Consulta **diariamente** a última *release estável* de `BruceDevices/firmware`, através da API pública GitHub; permite execução manual.
- Compara a tag mais recente com a tag incorporada em `COMANDO_RF_UPSTREAM.md`. Se coincidir, não cria aviso.
- Se existir atualização ainda não incorporada, abre **uma Issue por versão** em `martaxi-boss/Comando-rf`, atribuindo-a a `martaxi-boss`, com links para release, mudanças e binário ESP32-C5, se existir.
- Não cria issues duplicadas para a mesma versão, incluindo as já encerradas. As notificações por email/app dependem das preferências GitHub do proprietário.
- **Não** descarrega/instala firmware na placa, **não** altera o código automaticamente, **não** executa CI desconhecido do autor e **não** atualiza a VPS.
- Quando aprovada e testada uma nova versão, atualizar a cópia do código, documentar SHA/compatibilidade, testar e corrigir o registo de proveniência. O monitor passa então a comparar com a nova referência.
- GitHub Actions agendado é um serviço *best effort*: poderá haver atrasos; confirmar a primeira execução real em **Actions** antes de declarar o alerta operacionalmente validado.

## 8. Próximos testes sem atalhos

1. Confirmar referências do material recebido e alimentação sem curto-circuitos.
2. Instalar firmware oficial `esp32-c5-tft` e validar versão no ecrã.
3. Verificar ILI9341, toque XPT2046, microSD (se presente) e CC1101 (primeiro receção).
4. Medir a frequência do equipamento próprio e confirmar o protocolo, sem pressupor suporte de rolling code.
5. Só após autorização e validação do emparelhamento, ensaiar transmissão e persistência do contador, com plano de recuperação.
6. Reavaliar se a interface «Abrir portão» acrescenta valor face a RF Emulate existente.
7. Verificar o monitor via Action real, incluindo um cenário de versão nova sem alterar firmware.

**Estado de conclusão documental:** inventário, ligações e procedimentos estão documentados; testes de campo e binário personalizado permanecem pendentes.
