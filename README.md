# Hoopless Stack

## Installation

Install with [Git](https://git-scm.com/):
```shell script
git clone --recurse-submodules https://github.com/Ouxsoft/hoopless-stack.git
```

Start local development stack using [Docker](https://docs.docker.com/get-docker/).

```shell script
./stack start
```

View in web browser:

|  | URL | Username | Password
| :--- | :--- | :--- | :--- |
| Website | [http://localhost](http://localhost) | | |
| Database | [http://db.localhost](http://db.localhost) | | |
| Continuous Deployment | [http://cd.localhost/blue](http://cd.localhost/blue) | admin | admin |

### Via Shell

Start production stack using [Docker](https://docs.docker.com/get-docker/).

```shell script
./stack start prod
```

## Contributing
Hoopless is an open source project. If you find a problem or want to discuss new features or improvements
**please** create an issue, and/or if possible create a pull request. Contributing is easy with this
docker based development environment.