<pre>
Title: Transaction Signing via QR Codes
Author: Song Chenwen, Bither Team
Type: Standards Track
Created: 2014-9-6
</pre>

## Abstract

This BIP describes a way to sign Bitcoin transactions via QR Codes. 

Let us suppose we have two devices here. One can build and send transactions. And the other has the private key to sign transactions. 

We defined a format to send unsigned transactions via QR Codes to the later, and send signed transactions back to the former.

## Motivation

In this way, the Bitcoin private keys can be stored away from network connected devices. This is the safest way to store private keys. And via QR Codes, the user can still spend bitcoins belonging to these private keys. 

It is easier for different online and offline clients to communicate with each other, if they all follow one common protocol of QR Codes content format.

## Process

There are two roles in this signing process, "HOT" for the online device which has the private key's transaction history and unspent transaction outputs, "COLD" for the offline device which hold the private key.

1. HOT generats a transaction based on the unspent transaction outputs. But it can't sign the transaction.
2. HOT converts the information needing to be signed in the unsigned transaction into formated signable QR Codes. HOT displays the QR Codes.
3. COLD scans HOT's screen and gets the signable information.
4. COLD finds the right private key that the signable information requires.
5. COLD signs the signable information using the private key.
6. COLD converts the signed information to formated QR Codes and displays them.
7. HOT gets the signed information and attach it onto the unsigned transaction.
8. HOT verifies if the transaction is correctly signed. The transaction will be spread to other peers if it passes the verification. 

## Format

HOT and COLD can understand each other only by restricting the QR Codes they provide to a certain format protocol.

The protocol goes below.

### Unsigned Information

Since We need to prove we own the private key which the unsigned transactions inputs have been sent to, so we need the inputs to be signed. 

And COLD need some information to ask the user if he likes to sign the transaction.

Following parts are joined and seperated by colon to make an unsigned QR Codes content.

* The address used to send bitcoin from
* The transaction fee
* The address to receive bitcoin
* The amount of bitcoin being sent (not including transaction fee)
* Sha256 hash byte arrays of all the transaction inputs seperated by colons which is in hex string format

[source code](https://github.com/bither/bither-android/blob/master/bither-android/src/net/bither/model/QRCodeTxTransport.java#L152)

### Signed Information

After COLD signs the transaction's inputs, it builds the input scripts based on the ECDSA signactures.

COLD convert all the input scripts to hex strings and join them with colon as separator.

This joined string will be the content for the signed QR Codes.

[source code](https://github.com/bither/bither-android/blob/master/bither-android/src/net/bither/activity/cold/SignTxActivity.java#L140)

## Optimization for QR Code

According to [Information capacity and versions of the QR Code], one QR Code can serve more content if it only contains uppercased letters, digits and some kinds of punctuations, so we can optimize our content.

Following rules are applied to the content.

* All byte array hex strings only use uppercased letters ([source code](https://github.com/bither/bither-android/blob/master/bither-android/src/net/bither/util/StringUtil.java#L197))
* Bitcoin addresses are represented by the hex strings of the pre-base58 bytes.
* Amounts of bitcoins are represented by uppercased hex strings for the count of satoshis.


## QR Code Pagination

One transaction can contain multiple inputs. Therefore the content of unsigned or signed information can be too long to be displayed in one QR Code. 

A QR Code in a limited graphic size with too much information can be too hard to scan.

So we need a way to paginate the QR Codes.

The length of one QR Code's content can vary with different graphic size. We find 328 to be a suitable content length to be displayed and easily scanned on mobile phones.

When we need multiple pages of QR Codes. We add `(Total page count minus 1):(current page index starts from 0):` in front of every page. e.g. QR Code Page 1 in 3 pages will be prefixed with `2:0:`.

[source code](https://github.com/bither/bither-android/blob/master/bither-android/src/net/bither/util/StringUtil.java#L245)


[Information capacity and versions of the QR Code]: http://www.qrcode.com/en/about/version.html
