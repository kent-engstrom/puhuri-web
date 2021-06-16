# Integrating Identity Providers with Puhuri AAI

## Overview

Puhuri AAI uses MyAccessID platform as an integration point for all IdPs. MyAccessID is published in eduGAIN as a Service Provider and will consume IdPs available in eduGAIN through the metadata exchange mechanism. MyAccessID can on request integrate IdPs in the case they are not available in eduGAIN.

![Puhuri AAI](../assets/puhuri_aai.png )

## Integration of IdPs

Depending on whether IdP is available in eduGAIN, following possibilities exist:

- **IdPs that are available in eduGAIN** will be automatically integrated. Please refer to
  [Requirements for IdPs integrated through eduGAIN](requirements-edugain.md) for further
  specification on requirements. You can check if your IdP is registered at
  [https://release-check.edugain.org/](https://release-check.edugain.org/).

- **IdPs that are not available in eduGAIN** will be integrated on a request basis. National
  LUMI integration contacts should get in touch with Puhuri to request integration and exchange
  the integration end points. Please refer to [Requirements for directly integrated IdPs](requirements-direct.md)
  for further specification on requirements.


## Questions?

Contact person for additional information: marina at sunet.se.
