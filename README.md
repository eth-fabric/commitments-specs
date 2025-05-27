# Ethereum Commitments API Specifications

**These standards are currently under review / feedback and are not audited.**

This repo contains an open and neutral API specification for proposer commitments on Ethereum. This was developed via collaboration and contribution from teams across the Ethereum ecosystem who were interested in helping develop and enable proposer commitments on Ethereum. These efforts started at zuBerlin, continued through Edge City Sequencing Week, and have progresed through community calls (recordings can be found in the pm repo). 


### Render API Specification
To render spec in browser, you will simply need an HTTP server to load the index.html file in root of the repo.

For example:

```bash
git submodule update --init --recursive # don't forget the submodules!
python3 -m http.server 8080
```

The spec will render at http://localhost:8080.

### Disclaimer
This project is not audited and is not ready for production use. Please see [Terms of Use](https://github.com/Commit-Boost/commit-boost-client/blob/main/TERMS-OF-USE.md) for more details.
