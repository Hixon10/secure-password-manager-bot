# Password manager implemented as telegram bot

[This bot](https://t.me/SecurePasswordManagerBot) allows saving and retrieving passwords. 
It is expected that before saving a password, you will encrypt it via [https://secure-password-manager-bot-web.pages.dev/](https://secure-password-manager-bot-web.pages.dev/), 
using your master password. 

The bot is implemented on top of [Cloudflare Workers](https://workers.cloudflare.com/) and [Workers KV](https://www.cloudflare.com/products/workers-kv/).
The worker `AES-CBC` encrypts passwords, before saving to the `Workers KV`. Encryption key are stored inside [Cloudflare Secrets](https://blog.cloudflare.com/workers-secrets-environment/).
In other words, we encrypt every password two times. Firstly, you do it, using own master password. Secondly, CF worked does it, using shared secret.

## Credits
1. [https://github.com/my-telegram-bots/hitokoto_bot](https://github.com/my-telegram-bots/hitokoto_bot) - I use it as a telegram bot template.
2. [https://bitwarden.com/help/article/what-encryption-is-used/](https://bitwarden.com/help/article/what-encryption-is-used/) and [https://bitwarden.com/help/crypto](https://bitwarden.com/help/crypto) - I use it for cryptography-related code. 
3. [https://www.cloudflare.com/](https://www.cloudflare.com/) - I use it as free telegram bot hosting. 

