---
title: "自宅サーバ2: Discort bot"
date: 2023-10-06T17:57:25+09:00
draft: false
---

Heroku なにしてるん

## ubuntu server 導入

普通にやる．
下記を参考にした．

<https://qiita.com/yankee/items/495e80193070f6e70b65>

## Discord bot を導入する

とりあえず入門によいのではないかと思い，やってみる．

まず Discord bot への登録 (?) が必要．
手順どおりやれば OK (<https://discord.com/developers/docs/intro>)

次に Ubuntu 側での操作．
適当にディレクトリを作り，.env にトークンとチャンネル ID を記入する．
あとは良い感じにアプリを作る．
今回は，実行すると ctftime から今週開催される ctf を教えてくれるアプリを作った．
次は定期実行をやる

```python
import os
from datetime import datetime, timedelta
import aiohttp
import discord
from dateutil import parser as dtparser, tz
from dotenv import load_dotenv


# env
load_dotenv()
TOKEN       = os.getenv("DISCORD_TOKEN")
CHANNEL_ID  = os.getenv("CHANNEL_ID")
TIMEZONE    = os.getenv("TIMEZONE", "Asia/Tokyo")
CHANNEL_ID = int(CHANNEL_ID)
TZ         = tz.gettz(TIMEZONE)
API_EVENTS = "https://ctftime.org/api/v1/events/?limit=100&start={}&finish={}"


def _to_epoch(val: int | str) -> int:
    """CTFtime start/finish has int and ISO8601 -> epoch"""
    if isinstance(val, (int, float)):
        return int(val)
    s = str(val)
    if s.isdigit():
        return int(s)
    return int(dtparser.isoparse(s).timestamp())

def this_week_range() -> tuple[int, int]:
    now     = datetime.now(TZ)
    monday  = now - timedelta(days=now.weekday())
    monday  = monday.replace(hour=0, minute=0, second=0, microsecond=0)
    sunday  = monday + timedelta(days=7, seconds=-1)
    return int(monday.timestamp()), int(sunday.timestamp())

async def fetch_events(start_ts: int, end_ts: int) -> list[dict]:
    url     = API_EVENTS.format(start_ts, end_ts)
    timeout = aiohttp.ClientTimeout(total=20)
    async with aiohttp.ClientSession(timeout=timeout) as sess:
        async with sess.get(url) as resp:
            resp.raise_for_status()
            data = await resp.json()
    for ev in data:
        ev["start"]  = _to_epoch(ev["start"])
        ev["finish"] = _to_epoch(ev["finish"])
    return sorted(data, key=lambda e: e["start"])

def format_events(events: list[dict]) -> str:
    head = "**CTF this week**\n"
    if not events:
        return head + "no ctf..."

    lines = [head]
    for ev in events:
        st = datetime.fromtimestamp(ev["start"], tz=tz.UTC).astimezone(TZ)
        en = datetime.fromtimestamp(ev["finish"], tz=tz.UTC).astimezone(TZ)
        lines.append(
            f"? **{ev['title']}**\n"
            f"　{st:%Y-%m-%d %H:%M} ? {en:%Y-%m-%d %H:%M}\n"
            f"　<{ev['url']}>"
        )
    return "\n".join(lines)

# bot
class WeekBot(discord.Client):
    async def on_ready(self):
        print(f"Logged in as {self.user} (ID: {self.user.id})")
        try:
            channel = await self.fetch_channel(CHANNEL_ID)
        except discord.NotFound:
            print(f"no CHANNEL_ID")
            await self.close()
            return
        except discord.Forbidden:
            print(f"no right for CHANNEL_ID")
            await self.close()
            return

        start_ts, end_ts = this_week_range()
        events           = await fetch_events(start_ts, end_ts)
        await channel.send(format_events(events))
        await self.close()

def main() -> None:
    intents = discord.Intents.default()
    discord.utils.setup_logging()
    WeekBot(intents=intents).run(TOKEN)

if __name__ == "__main__":
    main()
```
