# Q3:TEAM ARENA Boomer Break

host your own web-based quake lan party.

## running

download the [docker-compose.yml](https://raw.githubusercontent.com/subjec2change/q3/refs/heads/main/docker-compose.yml) and run `docker compose up`.

visit <server-ip> and play.

to keep things simple every visitor will join the same lobby by default, but if you want a private game you can append `?server=whatever` or what ever the url is and share that link

don't have friends? no problem, activate bots by appending `?lonely` or use the setup button

want to play a different map from the demo? append `?map=q3dm7` (options are `q3dm1`, `q3dm7`, `q3dm17`, `q3tourney2`) or again use the setup button

## disclaimer

this is a very minimal scrape and remix of the extremely cool XanderStrike/shake who's source can be found [here](https://github.com/XanderStrike/shake) and [here](https://github.com/jdarpinian/HumbleNet)

inspired by how I wanted the full game to work with only IP's and not full domains and also to use caddy to serve the whole dam thing and it work in the office.
