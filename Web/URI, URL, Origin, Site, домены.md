
**URI** - общий термин для идентификатора ресурса. URL является разновидностью URI.

**URL** - конкретный адрес ресурса.

https://example.com:443/users?id=10

**Scheme** - протокол.

https

**Домен** - имя сайта или хоста.

example.com

**Поддомен** - часть перед доменом.

**Host** - хост, обычно домен или IP-адрес.

api.example.com

**Port** - порт.

https://example.com:443

                  ↑

                port

**Origin** - `scheme + host + port`.

https://api.example.com:443

└─scheme─┘ └────host─────┘ └port┘

Если отличается хотя бы одна из этих частей, origin другой.

**Site** - более широкое понятие, основанное на домене верхнего уровня и зарегистрированном домене. Например:

app.example.com

api.example.com

Это **разные origin**, но обычно **один site**: `example.com`.

### Как запомнить

URL
│
├── scheme       https
├── host
│   ├── subdomain  api
│   └── domain     example.com
├── port          443
├── path          /users
├── query         ?id=10
└── fragment      #profile

А для безопасности главное:

**Origin** → `scheme + host + port` → CORS

**Site** → доменная принадлежность → SameSite cookie

**URL** → полный адрес ресурса.
