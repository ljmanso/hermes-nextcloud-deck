---
name: nextcloud-deck
description: Nextcloud task deck integration into Hermes (**only the deck**).
version: 1.0.0
license: MIT
metadata:
  hermes:
    tags: [Nextcloud, Deck, Tasks]
---

# Nextcloud

This is an integration of Nextcloud task deck into Hermes (**only the deck**).

Requires: The Deck should be already working on Nextcloud

## Setup

The following environment variables should be added into your .env.

```bash
NEXTCLOUD_DECK_URL="https://nextcloud.myselfhost.com"
NEXTCLOUD_DECK_USERNAME="your_username"
NEXTCLOUD_DECK_PASSWORD="user_and_password"
```

## Usage

### Get deck boards

This function `get_deck_boards` returns a dictionary of the boards available in the Nextcloud deck. There should be a board named "ToDo", "to do list", "tasks", or something like that, wich the user should have already created. We will need to pay attention to the fields `title` and `id`.

```python
import requests
from requests.auth import HTTPBasicAuth
HEADERS = {"OCS-APIRequest": "true", "Content-Type": "application/json"}
NEXTCLOUD_DECK_URL = os.environ["NEXTCLOUD_DECK_URL"]
NEXTCLOUD_DECK_USERNAME = os.environ["NEXTCLOUD_DECK_USERNAME"]
NEXTCLOUD_DECK_PASSWORD = os.environ["NEXTCLOUD_DECK_PASSWORD"]

def get_deck_boards():
    """Fetch all available Deck boards."""
    url = f"{NEXTCLOUD_URL}/index.php/apps/deck/api/v1.0/boards"
    response = requests.get(url, auth=HTTPBasicAuth(USERNAME, PASSWORD), headers=HEADERS)
    response.raise_for_status()
    return response.json()
```


### Get deck stacks

This function gets the stacks in a given board (`board_id`). We can safely assume that we will use the first stack in the board.

```python
import requests
from requests.auth import HTTPBasicAuth
HEADERS = {"OCS-APIRequest": "true", "Content-Type": "application/json"}
NEXTCLOUD_DECK_URL = os.environ["NEXTCLOUD_DECK_URL"]
NEXTCLOUD_DECK_USERNAME = os.environ["NEXTCLOUD_DECK_USERNAME"]
NEXTCLOUD_DECK_PASSWORD = os.environ["NEXTCLOUD_DECK_PASSWORD"]

def get_deck_stacks(board_id: int):
    """Fetch all stacks (columns) in a board."""
    url = f"{NEXTCLOUD_URL}/index.php/apps/deck/api/v1.0/boards/{board_id}/stacks"
    response = requests.get(url, auth=HTTPBasicAuth(USERNAME, PASSWORD), headers=HEADERS)
    response.raise_for_status()
    return response.json()
```


### Create a tag in a board

Given a board identifier `board_id` and a tag title `tag_title`, we can create a tag in our board. To assign a tag, we may not need to create a tag identifier, as it may already exist. It's good to keep track of existing tags to prevent duplicating them.

```python
import requests
from requests.auth import HTTPBasicAuth
HEADERS = {"OCS-APIRequest": "true", "Content-Type": "application/json"}
NEXTCLOUD_DECK_URL = os.environ["NEXTCLOUD_DECK_URL"]
NEXTCLOUD_DECK_USERNAME = os.environ["NEXTCLOUD_DECK_USERNAME"]
NEXTCLOUD_DECK_PASSWORD = os.environ["NEXTCLOUD_DECK_PASSWORD"]

def create_tag_on_board(board_id: int, tag_title: str, color: str = "ffffff"):
    url = f"{NEXTCLOUD_URL}/index.php/apps/deck/api/v1.0/boards/{board_id}/labels"
    payload = {"title": tag_title, "color": color}
    response = requests.post(
        url, json=payload,
        auth=HTTPBasicAuth(USERNAME, PASSWORD),
        headers=HEADERS
    )
    response.raise_for_status()
    return response.json()
```

### Get date

We can use the function `get_nextcloud_date` to compute today's date, or days from today

```python
import datetime
def get_nextcloud_date(offset: int=0):
    return ( datetime.datetime.now()+datetime.timedelta(days=offset) ).strftime('%Y-%m-%dT%H:%M:%S')

```

### Create a card in the deck

Given our board and stack identifier we can create cards (also known as tasks). We will always be using the same board and stack identifiers; we figure which ones to use and then memorise them. In addition to the basic data, we need a title and a due date. If the user does not provide the due date, we need to ask. The date needs to be in the format "2026-06-01T00:00:00+00:00". The user may provide a description that we need to provide to the function.

If `due_date` is not specified (None), we take it as today. If it's an integer, we interpret it as an offset (number of days from today), otherwise, we take the date as it is..
 
```python
import requests
from requests.auth import HTTPBasicAuth
HEADERS = {"OCS-APIRequest": "true", "Content-Type": "application/json"}
NEXTCLOUD_DECK_URL = os.environ["NEXTCLOUD_DECK_URL"]
NEXTCLOUD_DECK_USERNAME = os.environ["NEXTCLOUD_DECK_USERNAME"]
NEXTCLOUD_DECK_PASSWORD = os.environ["NEXTCLOUD_DECK_PASSWORD"]

def create_deck_card(board_id:int, stack_id:int, title:str, due_date:Union[None,int,str], description:str=None):
    """Create a card in a specific Deck stack."""
    url = f"{NEXTCLOUD_URL}/index.php/apps/deck/api/v1.0/boards/{board_id}/stacks/{stack_id}/cards"
    payload = {
        "title": title,
        "description": description,
        "type": "plain",
        "order": 0,
    }

    # If the due_date is not specified (None), we take it as today. If it's an int, we interpret it as an offset (number of days from today), otherwise, we take it as the date.
    if due_date is None:
        due_date = today_date()
    elif type(due_date) is int:
        due_date = today_date(offset)

    payload["duedate"] = due_date  # Format: "2026-06-01T00:00:00+00:00"
    response = requests.post(url, json=payload, auth=HTTPBasicAuth(USERNAME, PASSWORD), headers=HEADERS)
    response.raise_for_status()
    return response.json()
```


### Assign label to a card

After creating a card, we can assign tags to it. The returned value is sometimes not very helpful. Try to verify that the tags have been assigned correctly by retrieving the cards.

```python
import requests
from requests.auth import HTTPBasicAuth
HEADERS = {"OCS-APIRequest": "true", "Content-Type": "application/json"}
NEXTCLOUD_DECK_URL = os.environ["NEXTCLOUD_DECK_URL"]
NEXTCLOUD_DECK_USERNAME = os.environ["NEXTCLOUD_DECK_USERNAME"]
NEXTCLOUD_DECK_PASSWORD = os.environ["NEXTCLOUD_DECK_PASSWORD"]

def assign_label_to_card(board_id: int, stack_id: int, card_id: int, label_id: int):
    """Assign a label/tag to an existing card."""
    url = f"{NEXTCLOUD_URL}/index.php/apps/deck/api/v1.0/boards/{board_id}/stacks/{stack_id}/cards/{card_id}/assignLabel"
    payload = {"labelId": label_id}
    response = requests.put(
        url, json=payload,
        auth=HTTPBasicAuth(USERNAME, PASSWORD),
        headers=HEADERS
    )
    response.raise_for_status()
    return response.json()
```



### Get board labels

We can get a dictionary of the labels available in a board.

```python
import requests
from requests.auth import HTTPBasicAuth
HEADERS = {"OCS-APIRequest": "true", "Content-Type": "application/json"}
NEXTCLOUD_DECK_URL = os.environ["NEXTCLOUD_DECK_URL"]
NEXTCLOUD_DECK_USERNAME = os.environ["NEXTCLOUD_DECK_USERNAME"]
NEXTCLOUD_DECK_PASSWORD = os.environ["NEXTCLOUD_DECK_PASSWORD"]

def get_board_labels(board_id: int):
    """Returns labels as a dict: {label_title: label_id}"""
    url = f"{NEXTCLOUD_URL}/index.php/apps/deck/api/v1.0/boards/{board_id}"
    response = requests.get(
        url,
        auth=HTTPBasicAuth(USERNAME, PASSWORD),
        headers={"OCS-APIRequest": "true", "Content-Type": "application/json"}
    )
    response.raise_for_status()
    board = response.json()
    return {label["title"]: label["id"] for label in board.get("labels", [])}
```



### Get a card by its identifier

This function retrieves a single card from our stack.

```python
import requests
from requests.auth import HTTPBasicAuth
HEADERS = {"OCS-APIRequest": "true", "Content-Type": "application/json"}
NEXTCLOUD_DECK_URL = os.environ["NEXTCLOUD_DECK_URL"]
NEXTCLOUD_DECK_USERNAME = os.environ["NEXTCLOUD_DECK_USERNAME"]
NEXTCLOUD_DECK_PASSWORD = os.environ["NEXTCLOUD_DECK_PASSWORD"]

def get_card(board_id: int, stack_id: int, card_id: int):
    """Fetch a single card by its ID."""
    url = f"{NEXTCLOUD_URL}/index.php/apps/deck/api/v1.0/boards/{board_id}/stacks/{stack_id}/cards/{card_id}"
    response = requests.get(
        url,
        auth=HTTPBasicAuth(USERNAME, PASSWORD),
        headers={"OCS-APIRequest": "true", "Content-Type": "application/json"}
    )
    response.raise_for_status()
    return response.json()
```


### Get a list of all pending cards

This function provides a list of the cards available in our stack that have not yet been completed.

```python
import requests
from requests.auth import HTTPBasicAuth
HEADERS = {"OCS-APIRequest": "true", "Content-Type": "application/json"}
NEXTCLOUD_DECK_URL = os.environ["NEXTCLOUD_DECK_URL"]
NEXTCLOUD_DECK_USERNAME = os.environ["NEXTCLOUD_DECK_USERNAME"]
NEXTCLOUD_DECK_PASSWORD = os.environ["NEXTCLOUD_DECK_PASSWORD"]
def get_all_cards(board_id: int):
    """Returns all pending cards across all stacks as a list of dicts."""
    url = f"{NEXTCLOUD_URL}/index.php/apps/deck/api/v1.0/boards/{board_id}/stacks"
    response = requests.get(
        url,
        auth=HTTPBasicAuth(USERNAME, PASSWORD),
        headers={"OCS-APIRequest": "true", "Content-Type": "application/json"}
    )
    response.raise_for_status()
    stacks = response.json()

    all_cards = []
    for stack in stacks:
        for card in stack.get("cards", []):
            done = card.get("done", False)
            if done is False or done is None:
                all_cards.append({
                    "id": card["id"],
                    "title": card["title"],
                    "description": card.get("description", ""),
                    "due_date": card.get("duedate"),
                    "labels": [label["title"] for label in card.get("labels") or []],
                })
    return all_cards
```



### Get a list of all pending cards that need to be completed in the following days

This function provides a list of the cards available in our stack that have not yet been completed, and need to be completed in the following `days` days.
If the user needs overdue tasks, we can use `days=-1`.
If the user needs the tasks to be completed by today, we can use `days=0`.
If the user needs the tasks to be completed by tomorrow, we can use `days=1`.
If the user needs the tasks to be completed by the next 2 days, we can use `days=2`, and so forth.

```python
import requests
from requests.auth import HTTPBasicAuth
This function provides a list of the cards available in our stack.
import datetime
def get_cards_in_days(board_id: int, days: int):
    all_cards = get_all_cards(board_id)
    ret = []
    compare = (datetime.datetime.now()+datetime.timedelta(days=1+days)).replace(hour=0, minute=0, second=0)
    for card in all_cards:
        d = datetime.datetime.strptime(card["due_date"], '%Y-%m-%dT%H:%M:%S+00:00')
        if d<=compare:
            ret.append(card)
    return ret
```



---

## Output Format

All commands return JSON:

```json
{"status": "success", "data": {...}}
{"status": "error", "message": "Descriptive error message"}
```

For list operations, `data` is always an array even if empty: `[]`.

---

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `NEXTCLOUD_DECK_URL` | Yes | Base URL of your Nextcloud instance (no trailing slash) |
| `NEXTCLOUD_DECK_USERNAME` | Yes | Your Nextcloud username |
| `NEXTCLOUD_DECK_PASSWORD` | Yes | App Password (from Settings → Security → App passwords) |

Credentials should have been stored in `~/.hermes/.env`.

---

## Rules for Hermes Agent

1. **Confirm before destructive operations** — always show what will be deleted before running a delete command
2. **Respect rate limits** — avoid rapid sequential API calls; batch reads where possible



