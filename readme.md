# Q3:TEAM ARENA Boomer Break

host your own web-based quake lan party.

## running

download the [docker-compose.yml](https://raw.githubusercontent.com/subjec2change/q3/refs/heads/main/docker-compose.yml) and run `docker compose up`.

visit <server-ip> and play.

to keep things simple every visitor will join the same lobby by default, but if you want a private game you can append `?server=whatever` or what ever the url is and share that link

don't have friends? no problem, activate bots by appending `?lonely` or use the setup button

want to play a different map from the demo? append `?map=q3dm7` (options are `q3dm1`, `q3dm7`, `q3dm17`, `q3tourney2`) or again use the setup button

## disclaimer

Based on [shake](https://github.com/xanderstrike/shake) by XanderStrike, which builds on:
- [ioquake3](https://github.com/jdarpinian/ioq3) by jdarpinian
- [HumbleNet](https://github.com/jdarpinian/HumbleNet) by jdarpinian
