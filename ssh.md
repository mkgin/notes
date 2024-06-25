# ssh

## client options 
* when you have some hosts at the same IP#, on different ports
 * ```-o "StrictHostKeyChecking no```

## ~/.ssh/config 

* helpful for tunnels
    * `ServerAliveInterval 60`

* Old Switches where you might not be able to update firmware
  * old cisco, hpe
  * ```KexAlgorithms +diffie-hellman-group1-sha1
    Ciphers +3des-cbc```
	
* Forwarding... if 
  * ```ForwardAgent yes```
