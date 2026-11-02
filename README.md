# nodes7

> **This is `@oriolrius/nodes7`, a fork of [`@st-one-io/nodes7`](https://github.com/st-one-io/nodes7)** with fixes for the S7comm fill bytes ("padding") that are still open upstream. It is a drop-in replacement: same API, only the package name changes.
>
> | Version | Changes compared to `@st-one-io/nodes7` 1.1.2 |
> |---|---|
> | 1.1.5 | Reads no longer return zeros in the first bytes of a variable that starts just before the point where a long variable was split across packets ([st-one-io/nodes7#20](https://github.com/st-one-io/nodes7/issues/20), reported by @Poseidonas). |
> | 1.1.4 | Fill bytes are counted only between items: reads no longer skip the first byte of an item at a part boundary, and writes that exactly fill the PDU are accepted again. |
> | 1.1.3 | Multi-item writes send the fill bytes again; 1.1.2 dropped them and PLCs answer `0x8104` ([st-one-io/nodes7#16](https://github.com/st-one-io/nodes7/issues/16)). `writeItems()` counts the fill bytes when splitting requests, so writing many booleans/bytes no longer exceeds the PDU ([#8](https://github.com/st-one-io/nodes7/issues/8)). The read optimizer reserves the response fill bytes, fixing `0x8500` with many odd-length variables ([#17](https://github.com/st-one-io/nodes7/issues/17)). Unit tests no longer depend on the local timezone; integration tests against a strict PLC simulator. |
>
> The fixes are proposed upstream; this fork exists until they are released there.
>
> ```bash
> npm install @oriolrius/nodes7
> ```

NodeS7 is a library that allows communication to S7-200/300/400/1200/1500 PLCs using the Siemens S7Communication protocol over ISO-on-TCP (RFC1006) or MPI/PPI/DP USB Adapters.

Niether the software nor the authors are affiliated with Siemens in any way. S7-200, S7-300, S7-400, S7-1200 and S7-1500 are trademarks of Siemens AG.

This is a complete rewrite of and inspired on the awesome work of [plcpeople/nodeS7](https://github.com/plcpeople/nodeS7).


## WARNING
Fully test everything you do. The authors, contributors or anything or anybody related can be hold liable for injuries caused by the usage of this software. In situations where writing to a random area of memory within the PLC could cost you money, back up your data and test this really well.  If this could injure someone or worse, consider other software.


## Usage Example

```js
const { S7Endpoint, S7ItemGroup } = require('nodes7');

let plc = new S7Endpoint({ host: '192.168.0.1', rack: 0, slot: 1 });

plc.on('error', e => console.log('PLC Error!', e));
plc.on('disconnect', () => console.log('PLC Disconnect'));
plc.once('connect', async () => {
	console.log('connected!');

	// check the documentation for all available functions
	await plc.getTime(); //gets the PLC's current date/time
	await plc.blockCount(); //gets a count of blocks from the PLC of each type

	// you can use the S7ItemGroup to perform optimized read/write of variables

	let itemGroup = new S7ItemGroup(plc);

	let vars = {
		TEST1: 'MR4', 			// Memory real at MD4
		TEST2: 'M32.2', 		// Bit at M32.2
		TEST3: 'M20.0', 		// Bit at M20.0
		TEST4: 'DB1,REAL0.20',	// Array of 20 values in DB1
		TEST5: 'DB1,REAL4',		// Single real value
		TEST6: 'DB1,REAL8',		// Another single real value
		TEST7: 'DB1,INT12.2'	// Two integer value array
	}
	itemGroup.setTranslationCB(tag => vars[tag]); //translates a tag name to its address
	itemGroup.addItems(Object.keys(vars));

	console.log(await itemGroup.readAllItems());

	await plc.disconnect(); 	//clean disconnection
});
```

## Documentation

Please check the [API documentation](doc/API.md), specially the [S7Endpoint](doc/API.md#S7Endpoint) and [S7ItemGroup](doc/API.md#S7ItemGroup) sections
## Tests

```bash
npm test                    # unit tests
npm run test:integration    # against a strict S7 PLC simulator (needs uv: https://docs.astral.sh/uv/)
```

The integration tests start `test/integration/s7-sim.py`, a python-snap7 server that, like a real CPU, applies every item of a multi-item write, requires the S7comm fill bytes and refuses requests or responses larger than the negotiated PDU. `NODES7_DIR` runs them against another build of the library.
