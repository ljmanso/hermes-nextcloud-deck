# hermes-nextcloud-deck

This is an integration of Nextcloud task deck into Hermes (*i.e.*, **only the deck**).

I tried [adnw-vinc/hermes-nextcloud](https://github.com/adnw-vinc/hermes-nextcloud) and it didn't work for me. Given that I'm only interested in the deck, I made my own. The Nextcloud API calls have been vibecoded, so, although it seems to work well, I would recommend using a separate Nextcloud account that doesn't have other critical stuff (*i.e.*, **use it at your own risk**).


## Installation

### Download the skill

```bash
cd ~/.hermes/skills/productivity
git clone https://github.com/ljmanso/hermes-nextcloud-deck nextcloud-deck
```


### Add the following environment variables into the `.env` file.

```bash
NEXTCLOUD_DECK_URL="https://nextcloud.yourserver.xyz"
NEXTCLOUD_DECK_USER="user"
NEXTCLOUD_DECK_PASSWORD="passwordortoken"
```


## Contributing

I'm happy to incorporate suggestions. Drop me a PR.


## License

MIT License. See [LICENSE](LICENSE) for the full text.
