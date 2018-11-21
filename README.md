ZCF on Electron Cash: A short walk-through
==========================================

This is a series of screenshots showing how the *work-in-progress* zero confirmation contracts currently work on Electron Cash.
This is the implementation available at the time of this writing in this pull request here:

https://github.com/Electron-Cash/Electron-Cash/pull/964

Preparations
============
Check out the code from the pull request:

```
$ git clone https://github.com/awemany/Electron-Cash
$ cd Electron-Cash
$ git checkout -b zero-conf-forfeits origin/zero-conf-forfeits
```

And then start Electron Cash to create a new default wallet on testnet:

```
$ ./electron-cash --testnet
```

Note that this all currently only works with Electron Cash's default
deterministic wallet mode. ZCF has not been tested at all with different
wallet modes and it should be assumed that it is broken in these scenarios.

**WARNING**: Like also mentioned in the pull request, everything here is preliminary
and **should only be tried on testnet or with a miniscule amount of mainnet coins**!
I take absolutely no responsibility whatsoever for money lost due to using this code.

Receive some testnet coins, so that your balance is strictly positive and
is also confirmed in a block:
![Start up](https://user-images.githubusercontent.com/13838274/48863433-a3f8c880-edc9-11e8-9775-7c1bc4db9f3e.png)


Using forfeits
==============
Let's go and send ourselves some money that includes a ZCF. Take an address
from the receiving addresses lib from the `Addresses` tab to the clipboard and
go the the send tab and paste it there, and enter an appropriate amount, which
needs to be a bit *less than half of the money available in the confirmed input* (as
the sent money will be secured against scamming with a currently fixed 1:1
forfeit):

The PR adds something new, the "Use zero-conf forfeit" checkbox. Activate it and
it should look about like this:
![Sending with forfeit](https://user-images.githubusercontent.com/13838274/48863443-af4bf400-edc9-11e8-8e7b-33d8c09af459.png)

Now click on `Preview` and you should end up with a forfeit-including transaction:
![Forfeit transaction](https://user-images.githubusercontent.com/13838274/48863454-b70b9880-edc9-11e8-802e-e070c2e598ae.png)

As one can see, the forfeit transaction includes an `OP_RETURN` marker field
that is currently set to the string ```ZCF\x00``` and meant to be used for
versioning the to-be-specified ZCF protocol between wallets.

It furthermore includes the forfeit output itself, of course. In the above
picture, it is the P2SH output which starts with a `p` in the new `cashaddr`
address formatting.  It is also shown with a light blue background, indicating
that it is a forfeit that I as the sender can spend forward (like an ordinary
change output). If it is a transaction that we receive which includes a
forfeit, the forfeit address would be shown in pink. The amount of the forfeit
is currently set to 1:1 the send money amount (excluding change back to
oneself).

For technical reasons (detecting the forfeit address and that a forfeit P2SH
contract is being used) this change output is mandatory in the current
implementation.

We can now go and sign and broadcast this transaction. The picture we will arrive at
will look about like this:
![Receiving the transaction with forfeit](https://user-images.githubusercontent.com/13838274/48863469-becb3d00-edc9-11e8-8448-facd0a5ae556.png)

What you can see now is that the incoming money (which is in this also the outgoing
money as we're staying in the same wallet for simplicity) is marked with a green `F`
to signify that it is protected by an appropriate forfeit.
This green `F` will change into the usual partial clock symbols when this transaction
becomes confirmed in a block.

If we go to the addresses tab, we can see that there is a third type of address listed now,
the forfeit P2SH addresses:

![Forfeits addresses on the address tab](https://user-images.githubusercontent.com/13838274/48863476-c68ae180-edc9-11e8-9c37-182918668cc5.png)

Of course we don't trust all this funny new stuff yet, so lets go and quickly
mop up all the outputs into an ordinary P2PKH output by disabling the
zero-conf forfeit checkbox. That transaction would look like this, for
example:
![Spending forfeits forward](https://user-images.githubusercontent.com/13838274/48863487-cbe82c00-edc9-11e8-9baf-a393fee20888.png)

This particular transaction can also be seen on the testnet explorer, for example here:

https://tbch.blockdozer.com/tx/b8a058345e94528d63c2408c28d8fc3ca44fd3a12c76a346fa1b9a09dcef173b

You can see that this actually spends the special forfeit P2SH by producing the
correct pushes to the stack, including the forfeit P2SH script.

Have fun!