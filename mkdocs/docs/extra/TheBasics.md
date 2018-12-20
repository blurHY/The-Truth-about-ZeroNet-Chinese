## Introduction

Before we start creating zites, it is useful to know how zites work on ZeroNet. This tutorial will take you through how to create an empty zite, where to put your code, what you need before you can publish your zite, and how they are updated, transfered, and kept secure.

Zites are made up of basic web code, including html, css, and js. Additionally, json files are used for storing data and for telling ZeroNet what databases you have and how they are structured.

Since ZeroNet makes no use of a server - everything is downloaded to your computer and ran locally - there is no way to do server side scripting. However, this is remedied by using the ZeroNet API called ZeroFrame. This API lets you query server/site/user info, load or modify files using a WebSocket connection to your ZeroNet client, and query and modify databases. [^1]

## Creating A Zite

There are currently two ways of creating a zite. You can clone an existing zite, or you can create your own zite from scratch. If you decide to clone a zite, there are many of them you can choose from, including [ZeroBlog](/Blog.ZeroNetwork.bit/). However, if you want to create your own zite from scratch, that is what the next few tutorials are going to cover. [^2]

In order to clone a zite, find a zite you want to clone, visit the zite so that it is downloaded to your computer. Next, go back to ZeroHello and find the zite in the sidebar. Click the menu icon, and press 'Clone'.

To create an empty zite, go to ZeroHello, click the menu icon at the top left next to the ZeroHello logo, and click 'Create new, empty site'. Once you click on this, you will be redirected to your new empty zite. Take note of its address, this is the address people will use to visit your zite. An empty zite will setup everything necessary for you to start creating a zite from scratch.

## The Sidebar

The Sidebar is _the_ place, where you can view information about any zite. It can be accessed by visiting a zite, and pulling the transluscent circle with the `0` in the middle, to the left. Here you can see an interactive globe that you can rotate, which displays all the peers over the globe.
There is a list of different items under it:
1. **Peers**:
Shows to how many peers you are connected, how many there are to which you can connect, how many use Onion, and the total amount of peers
2. **Data Transfer**: Shows how much data you have sent and received on the zite opened
3. **Files**: Shows what the summed size of the different file-formats are
4. **Size Limit**: Shows, how much of the filesize-limit you are using (in %), what the available free space is, on your harddrive, and an input to set the zites filesize-limit
5. **Optional Files**: Shows how much data you have downloaded, of all the optional files on the zite
6. **Download and help distribute all files**: Switch, to toggle if you want to download and seed _all_ the optional files
7. **Database**: Shows how big the database is, the path to it (relative to the zites path), a button to reload the Database-Scheme, and a button to rebuild the database
8. **Identity Address**: Shows how much storage you filled on the zite, of which the total is set by the Owner, and your Authentication-Address, with a button to select another account
9. **Site Control**: The first button tries to fetch the newest version available to any connected peer, the second pauses the download of any new data, and the third deletes the complete zite from the harddrive
10. **Site Address**: Displays the zites address (this might be helpful for finding the zite in your data-dir, if you are accessing it via a `.bit`-address)
11. **Missing Files**: Shows a list of all files, that are missing, according to the ZeroNet-Client
12. **This is my site**: Switch, to toggle, if you own this zite. If this is active, a few other items will appear underneath

The List-part **This is my site**:
1. **Site Title**: Input to change the title of the zite
2. **Site Description**: Input to change the description of the zite
3. **Save site settings**: A button, to save the last two inputs to the zites `content.json`-File
4. **Content Publishing**: A list of files you can sign (click them to insert them into the input below), an input to input the file you want to sign/publish, a button to sign the selected file, and another one, to publish it to connected peers

## Where Is A Zite Stored Locally?

If you need to change any of the code for your zite, you can find your zite in the `ZeroNet_Root/data` folder, under the directory of your zite address. This is the same as the `site address` shown in the sidebar. When you first start out, this directory should have a `content.json` file, an `index.html` file, and a `js` director where the `ZeroFrame.js` file is located.

## Content.json File, What is It?

When a visitor visit's your zite, the first thing they download is the zite's `content.json` file. This `content.json` file holds all of the other file names which must be downloaded in order to use the zite, along with their hashes so it can be verified that they are downloading the correct files. It basically gives information on the *content* of the zite. This file also holds a cryptographic digital signature that was created by the owner to verify that the content.json file has not been tampered with by anyone other than the zite owner. 

The file does contain other information like what ZeroNet version the zite was created for, the modified number to tell what version of the zite you are downloading, the zite's title and description, the address from which you cloned the zite, and any other important information.

## Signing the Content.json File

Whenever you make any changes to your zite, including to the `content.json` file, you must *sign* the file. This ensures that whenever a person downloads your zite, they know these changes were made by the zite owner. Signing your `content.json` file will update the hashes to all of the files stored and used on your zite.

**NOTE:** If you do not sign this file, visitors will not be able to download this updated version of your zite for security reasons. If you have no peers, they will not be able to visit your zite at all since they can't download a *signed* `content.json` file elsewhere.<br>
**So make sure you sign this file before you publish it.**

To sign this file, you go to the sidebar and, in the `This is my site` section, click `Sign`.

## How Zites Are Kept Secure

ZeroNet secures zites by using private and public keys, the public key being your zite's address. The private key is always kept on your computer, in the `users.json` file under the `ZeroNet_Root/data` directory. This private key is used to create a digital signature and encrypt the hash of the `content.json` file in its current state (excluding the digital signature stored in the file). This digital signature (and encrypted hash) is then stored in this file. When a visitor downloads the file, they first decrypt the digital signature with the zite's public key (the zite's address). Then, they verify the file by comparing the hash of the file with the hash that has been decrypted and sent via the digital signature.

This keeps your zite secure because the only person who can create the digital signature that holds the hash of the `content.json` file is the person with the private key - the zite owner.

## Publishing Your Zite

In order to publish your zite, you simply sign your `content.json` file and click `publish` in the sidebar. These buttons are located in the sidebar under the `This is my site` section. If you do not have any peers, you do not need to click publish. Publishing sends the `content.json` file to other peers. Without peers, there is no need because these peers will download your zite from your computer when they go to visit your zite.

## Keeping Your Zite Online

In order to keep your zite online, you need peers that are reliable and always on. If your computer will not be connected to ZeroNet all the time, you will have to either have enough peers to ensure your peer count won't go down to zero when you are offline, or you will have to find *at least* one peer that will always be online.

It may be a good idea to seed your zite by ZeroNet proxies, for example [bit.surf](http://bit.surf:43110). To do this, you simply visit your zite using the proxy of your choosing. This will help keep your zite online by having another computer seed the zite. [bit.surf](http://bit.surf:43110) is a good choice if you want your zite picked up by ZeroNet search engines like [RVRE](/rvre.bit).

You can also add your zite to [New ZeroNet Sites](/1LtvsjbtQ2tY7SCtCZzC4KhErqEK3bXD4n), which will not only further spread your zite, but also get seeders who seed from this list of new zites.

And finally, you will also want to add your zite to [ZeroSites](/Sites.ZeroNetwork.bit) so it can more easily be discovered.

## Updating Your Zite

Updating your zite is pretty simple. After making any changes to the zite, you must first sign your zite. This will update the hashes and the digital signature in the `content.json` file. After you have done that, click `publish`. This will send the new `content.json` file to a max of 6 different peers. These peers check if the file is newer than the one they already have. If it is, they download the rest of the files *that have been changed/updated* (by comparing the hashes in the new `content.json` file to the hashes in the old `content.json` file to see if they have changed).

After a peer has the updated files, they then send them to other peers using the same method as above. Eventually, all peers will have the updated files.

Note that before a peer download's the changed files, they first verify that the `content.json` file was modified by the zite owner by comparing the file's hash to the decrypted hash from the Digital Signature. And after they have download the updated files, they also verify them with the hashes in the `content.json` file.

---
Next Tutorial: [Creating A Static Zite](./?/tutorials/static_zite)<br>
Additionally: [Tips & Tricks Tutorial](./?/tutorials/tips_tricks)

[^1]: There is a page on the ZeroNet Read The Docs that gives a reference to the ZeroFrame API [here](/1Docsfk5uLvuXkL4U13eYDxCm2gzi1fNQq/site_development/zeroframe_api_reference/).
[^2]: There's also [this tutorial](/blog.zeronetwork.bit/?Post:43:ZeroNet+site+development+tutorial+1) from ZeroBlog.