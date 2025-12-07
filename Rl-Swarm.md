# Gensyn RL-Swarm Node - VPS 

## Системные требования

### Минимальные требования
- **ОС** Ubuntu 22.04 LTS
- **RAM** 16GB (рекомендуется 32GB)
- **CPU** 4 ядра (рекомендуется 8 ядер)
- **SSD** 50GB свободного места
---

## Быстрая установка (для опытных или ленивых)

### Установка одной командой
```bash
sudo apt update && sudo apt install -y python3 python3-venv python3-pip curl wget screen git lsof build-essential gcc g++ && git clone https://github.com/gensyn-ai/rl-swarm.git && cd rl-swarm && python3 -m venv .venv && source .venv/bin/activate && sed -i -E 's/(num_train_samples:\s*)2/\1 1/' code_gen_exp/config/code-gen-swarm.yaml && screen -S gensyn && ./run_rl_swarm.sh
```

---

### Настройка туннеля для логина

Откройте **новую SSH сессию** и выполните:
```bash
wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb && sudo dpkg -i cloudflared-linux-amd64.deb && cloudflared tunnel --url http://localhost:3000
```

Скопируйте полученную ссылку, откройте её в браузере и завершите вход.

---

**После окончания загрузки увидите следующее:**
1. Push to Hugging Face Hub? → Введите `N` и нажмите Enter
2. Model name? → Введите `Gensyn/Qwen2.5-0.5B-Instruct` и нажмите Enter
3. Prediction Market? → Введите `n` и нажмите Enter

**Отключиться от screen**  `Ctrl+A` + `D`



## Пошаговый гайд


### Шаг 1. Установка пакетов и зависимостей
```bash
sudo apt update && sudo apt upgrade -y
```
```bash
sudo apt install -y python3 python3-venv python3-pip curl wget screen git lsof nano unzip build-essential gcc g++
```

**Проверяем установку**
```bash
python3 --version  
git --version
screen --version
```

---

### Шаг 2.Клонируем репозиторий
```bash
cd ~
git clone https://github.com/gensyn-ai/rl-swarm.git
cd rl-swarm
```

---

### Шаг 3. Создаём виртуальное окружение
```bash
python3 -m venv .venv
source .venv/bin/activate
```

### Шаг 4. Настройка параметров обучения
```bash
sed -i -E 's/(num_train_samples:\s*)2/\1 1/' code_gen_exp/config/code-gen-swarm.yaml
```
```bash
grep "num_train_samples" code_gen_exp/config/code-gen-swarm.yaml
```


**Почему это важно:** Использование `train_samples: 1` снижает использование RAM с 12-15GB до 8-10GB и значительно уменьшает вероятность падения ноды.

---

### Шаг 5. Запуск ноды

**Запуск в screen сессии**
```bash
screen -S gensyn
```

**Запуск ноды**
```bash
./run_rl_swarm.sh
```


**Дождитесь следующего результата:**
```
>> Waiting for modal userData.json to be created...
```

Важно . Не закравая первую SSH сессию, откройте новую 

```bash
ssh root@YOUR_VPS_IP
```



### Установка туннеля


**Установите cloudflared.**
```bash
wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb
```

**Проверяем установку. **
```bash
cloudflared --version
```

---

### Шаг 2. Создание туннеля
```bash
cloudflared tunnel --url http://localhost:3000
```

После этой команды вы увидите ссылку в пунктирной рамке.

### Шаг 3: Завершение входа в браузере

1. Откройте полученную ссылку в браузере
2. Нажмите **"Sign in with Gmail"** (НЕ "Connect with Google")

3. Введите ваш email адрес
4. Проверьте почту на которую должен прийти код подтвержения
5. Введите код на странице входа
6. Нажмите Submit

После того как вы успешно залогинитесь , в первой сессии продолжится установка.
после успешной установки вам будут предложены вопросы

**Ответы на запросы:**
- `Would you like to push models to Hugging Face Hub? [y/N]` → `N`
- `Enter the name of the model` → `Gensyn/Qwen2.5-0.5B-Instruct`
- `Prediction Market? [y/N]` → `n`


### Альтернативные варианты туннеля


**Вариант A - SSH локальный проброс портов**

На вашем локальном пк
```bash
ssh -L 3000:localhost:3000 root@IP_ВАШЕГО_VPS
```

Затем откройте в браузере: `http://localhost:3000`

Держите этот терминал открытым во время входа.

**Вариант B - localhost.run**
```bash
ssh -o StrictHostKeyChecking=no -R 80:localhost:3000 nokey@localhost.run
```

**Вариант C - serveo**
```bash
ssh -R 80:localhost:3000 serveo.net
```



---

**Проверьте, что файлы созданы:**
```bash
ls -la ~/rl-swarm/userData.json
ls -la ~/rl-swarm/swarm.pem
```
Важно: Сохраните эти файлы где-нибудь в безопасном месте на вашем ПК

После этих шагов можно закрыть терминал с туннелем (Ctrl+C)


**Проверка логов ноды:**
```bash
screen -r gensyn
```

**Отключиться:** `Ctrl+A+D`

---




## Получение Swarm роли Discord

### Предварительные требования
- Нода работает непрерывно 24+ часа
- Peer ID зарегистрирован в блокчейне
- Один и тот же email используется для входа, дашборда и Discord

---

### Шаг 1: Установка Go
```bash
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.22.4.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
```

**Добавить в PATH**
```bash
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source $HOME/.bash_profile
```

**Проверяем установку**
```bash
go version
```

---

### Шаг 2. Установка gswarm
```bash
go install github.com/Deep-Commit/gswarm/cmd/gswarm@latest
```

**Проверяем установку**
```bash
gswarm --version
```

---

### Шаг 3. Создаём Telegram бота

**Откройте Telegram и найдите @BotFather:**

1. Отправьте `/newbot`
2. Выберите имя для бота (например, "My Gensyn Bot")
3. Выберите username (должен заканчиваться на "bot", например, "mygensyn_bot")
4. Скопируйте и сохраните полученный токен бота

---

### Шаг 4. Получаем свой Telegram Chat ID

**Найдите @userinfobot в Telegram:**

1. Начните разговор
2. Отправьте `/start`
3. Скопируйте ваш Chat ID 
---

### Шаг 5. Получение вашего EOA адреса

1. Перейдите на https://dashboard.gensyn.ai
2. Войдите с тем же email, который использовали для входа в ноду
3. Скопируйте ваш EOA адрес (начинается с 0x...)

---

### Шаг 6. Запуск gswarm
```bash
gswarm
```

**Введите при запросе:**
- Bot Token: Вставьте токен вашего бота от BotFather
- Chat ID: Вставьте ваш Chat ID от userinfobot  
- EOA Address: Вставьте ваш EOA из дашборда

**Если вы видите ошибку "No peer IDs found":**
- Подождите 1-2 часа (ноде нужно время для регистрации в блокчейне)
- Проверьте логи: `grep "is already registered" ~/rl-swarm/logs/swarm_launcher.log`
- Если вы видите ваш Peer ID зарегистрированным, попробуйте gswarm снова

---

### Шаг 7. Коннект Discord и Telegram

**В Discord:**
1. Перейдите на сервер Gensyn, канал `#swarm-link`
2. Введите: `/link-telegram`
3. Скопируйте предоставленный код подтверждения

**В Telegram:**
1. Откройте вашего бота (которого вы создали)
2. Отправьте: `/verify ВАШ_КОД_ЗДЕСЬ`
3. Дождитесь подтверждения
Вуаля!
 У вас должна появиться роль Swarm

