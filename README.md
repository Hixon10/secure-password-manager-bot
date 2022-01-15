# Password manager implemented as telegram bot

[This bot](https://t.me/SecurePasswordManagerBot) allows saving and retrieving passwords. Basically, it is a simple password manager.
It is expected that before saving a password, you will encrypt it via [https://secure-password-manager-bot-web.pages.dev/](https://secure-password-manager-bot-web.pages.dev/), 
using your master password. 

The bot is implemented on top of [Cloudflare Workers](https://workers.cloudflare.com/) and [Workers KV](https://www.cloudflare.com/products/workers-kv/).
The worker `AES-CBC` encrypts passwords, before saving to the `Workers KV`. Encryption key are stored inside [Cloudflare Secrets](https://blog.cloudflare.com/workers-secrets-environment/).
In other words, we encrypt every password two times. Firstly, you do it, using own master password. Secondly, CF worker does it, using shared secret.

## Credits
1. [https://github.com/my-telegram-bots/hitokoto_bot](https://github.com/my-telegram-bots/hitokoto_bot) - I use it as a telegram bot template.
2. [https://bitwarden.com/help/article/what-encryption-is-used/](https://bitwarden.com/help/article/what-encryption-is-used/) and [https://bitwarden.com/help/crypto](https://bitwarden.com/help/crypto) - I use it for cryptography-related code. 
3. [https://www.cloudflare.com/](https://www.cloudflare.com/) - I use it as free telegram bot hosting. 

## How to save password
1. Firstly, you need to encrypt your password, using [https://secure-password-manager-bot-web.pages.dev/](https://secure-password-manager-bot-web.pages.dev/). After that, you need to copy `Encrypted password`.
2. To save password, you need to use `/save` command with arbitrary password name and copied encrypted password. 

## How to retrieve password
1. Firstly, you need to get encrypted password via `/get` command. 
2. After that, you need to decrypt the password, using [https://secure-password-manager-bot-web.pages.dev/](https://secure-password-manager-bot-web.pages.dev/).

## Available bot commands:
1. **save** - `/save PASSWORD_NAME ENCRYPTED_PASSWORD`
2. **get** - `/get PASSWORD_NAME`
3. **mypasswords** - Get list of your passwrods
4. **deletepasswords** - Delete your passwords
5. **help** - How to use the bot
6. **about** - Info about bot

## How to self host the bot.
1. Firstly, you need to create [Workers KV](https://www.cloudflare.com/products/workers-kv/), which will be used as password storage. 
2. Secondly, you need to create [Worker](https://workers.cloudflare.com/) and copy a code from this repo. 
3. Thirdly, you need to set up previously created storage (step 1) for the Worker. You could do this either via [dashboard](https://dash.cloudflare.com), or using [Wrangler](https://developers.cloudflare.com/workers/cli-wrangler).
4. After that, you need to configure the following Secrets for your worker: `BOT_TOKEN` (you can get it via [@BotFather](https://t.me/BotFather)), `MASTER_PASSWORD` (crypto random string), `MASTER_PASSWORD_SALT` (crypto random string). Again, you can use either the dashboard, or CLI. 
5. Finally, you need to set up `Telegram Bot Webhooks` for created worker - `curl -F "url=https://<YOURDOMAIN.EXAMPLE>/<WEBHOOKLOCATION>" https://api.telegram.org/bot<YOURTOKEN>/setWebhook`

## Telegram bot authentication
Unfortunately, there is no way to correctly implement authentication for a telegram bot. Their [documentation](https://core.telegram.org/bots/api#setwebhook) says:

> If you'd like to make sure that the Webhook request comes from Telegram, we recommend using a secret path in the URL, e.g. https://www.example.com/<token>. Since nobody else knows your bot's token, you can be pretty sure it's us.

> If you want to limit access to Telegram only, please allow traffic from 149.154.167.197-233 (starting July 2019 please use: 149.154.160.0/20 and 91.108.4.0/22). Whenever something stops working in the future, please check this document again as the range might expand or change.


For me, it looks super unreliable. Instead, I would expect to have `mTLS`. Unfortunately, we don't have such thing at present. Therefore, we must use secret path for `Cloudflare worker`, and check client IPs:
```javascript
if (!(inSubNet(clientIP, '149.154.160.0/20') ||
	inSubNet(clientIP, '91.108.4.0/22'))) {

	// https://core.telegram.org/bots/webhooks#an-open-port
	console.log("[ERROR] received request from non telegram IP: ", clientIP)
	return new Response('ok: ', {status: 200})
}
```