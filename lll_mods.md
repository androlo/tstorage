New instructions added to LLLC. Modified compiler can be found at https://github.com/androlo/solidity, on the 'tstore' branch. Include `-DLLL=ON` when running cmake. 

Informally:

```
TLOAD cAddr sAddr
```

`TLOAD` reads the data stored at address `sAddr` in the transient storage of the account with address `cAddr`.

Example: if the account with address `0x00...01` wants to read from its own transient storage at address `0x20`, in LLL that would be `(TSTORE 0x00...01 0x20)`

```
TSTORE sAddr val
```

Stores the 32 byte value `val` at address `sAddr` in the account's own transient storage.


```
TCOPY cAddr sAddr mAddr len
```

Copies `len` bytes of data from the address `sAddr` in the transient storage of account `cAddr` to memory address `mAddr`.


#### Usage example

```
{
	(def "T_INIT_ADDR" 0x00)
	(def "T_MSG_ADDR" 0x20)

	(def "tInit" (TLOAD (ADDRESS) T_INIT_ADDR))
	(def "tMsg" (TLOAD (ADDRESS) T_MSG_ADDR))

	(def "tInitW" (val) (TSTORE T_INIT_ADDR val))
	(def "tMsgW" (val) (TSTORE T_MSG_ADDR val))

	(def "StaticInit" 
	    (unless tInit {
		(tInitW 0x01)
		(tMsgW 0x00)
	    })
	)

    StaticInit
	
	[0x20] (create {
	
		StaticInit

		(returnlll {
			StaticInit
			(when (= (TLOAD (CALLER) T_MSG_ADDR) "Here's ur message.") (tMsgW "Thanks, bro."))
			(return 0)
		})
	})

	(tMsgW "Here's ur message.")
	(msg @0x20 0)
	[[0x00]] (TLOAD @0x20 T_MSG_ADDR)
}
```

This contract does basic message passing without utilizing calldata or return data. It also performs a static initialization in the body, as well as in both init and body of the deployed contract, to make sure that transient variables are properly initialized.