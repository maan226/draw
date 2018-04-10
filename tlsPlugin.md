## TLS Plugin Deployement.
### Setup Guide

Unlike the usual setup use cases for EC, the TLS plugin can be used in scenarios where the connectivity between the EC Server agent and the data-source has to be encrypted. The most common use case is connecting to an end point ( data-source) using the "https" protocol over the port 443 instead of a regular "http" connection.

4. Create directories in the following pattern:
                EC
                |_ ec-gateway-agent
                                    |_ ecgaent_linux_sys
                                    |_ ec.sh
                                    |_ manifest.yml
                |_ ec_server_agent
                                    |_ ecgaent_linux_sys
                                    |_ ec.sh
                                    |_ manifest.yml
                                    |_ plugins.yml
                                    |_ tls_linux_sys
                 |_ ec-gateway-agent
                                    |_ ecgaent_OS_sys
                                    |_ ec.sh
                                    
