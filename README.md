### What is PixelMap.io?

A little over a decade ago, a website called MillionDollarHomePage.com was created by Alex Tew.  The page consisted (and still consists) of a 1000x1000 pixel grid, with a total of 1,000,000 pixels being sold for $1 each.  Because the pixels themselves were too small to be seen individually, they were sold in 10x10 pixel tiles for $100 each.  The purchasers of each tile then provided a 10x10 pixel image to be used, as well as optionally a URL for the tile to link to.  Once an image had been configured for a given tile however, it could never be changed (nor could the ownership of the tile).

In many ways, **PixelMap.io** is similar to the MillionDollarHomepage.  There are a total of 1,016,064 pixels for sale (on a 1,296 x 784 grid).  This grid is broken up into 3,969 16x16 tiles, each at an initial price of 10 Ethereum.  However, unlike the MillionDollarHomepage.com, every single tile is controlled by a single contract on the Ethereum Blockchain.  This lends itself to the following benefits.

* Each tile is truly owned by the entity that purchases it.  Because the data is stored on the Blockchain, nothing short of every single Ethereum node shutting down can eliminate the data.
* The contract is designed so that in the event that a tile owner would like to update the image, change the URL the tile points to, or sell the tile for any amount they'd like, they can, without any central authority facilitating or controlling any part of the process.
* In the event that PixelMap.io itself were to ever go down, the data, owner, and URLs for every single pixel remains on the Blockchain, and any site could easily replicate and display the overall image.  Essentially, the backend and database of the PixelMap.io is invincible as long as the Blockchain exists.  Additionally, we are looking into using Golem and/or IPFS to fully distribute the frontend as well.
* The project is completely Open Source, which means anyone can view the code, audit the Solidity contract, or even set up more frontends if they'd like.  For instance, if someone wanted to set up an easier-to-use tile editor for PixelMap.io, they could, as all of the data is stored safely on the Ethereum blockchain.

### PixelMap.io Solidity Contract

All pixel data for PixelMap.io is stored in 4 mapping variables, one for storing the owner, image, url, and price, for every single tile.  Each tile is initially unowned, and is purchasable for 10 Ethereum.  Once purchased, (using the buyTile function), the owner is set, and the price of the tile is set to 0.  A tile with a price of 0 cannot be purchased, and the price can only be changed by the owner of the tile.  The setTile function allows an owner to update the image, URL, and price of any tiles that they own.  If the price is set to a number above 0, then it essentially is up for sale, with the sale price going to the original owner.

Each 16x16 tile image is stored as a 768 character hexadecimal string.  The color of each pixel is determined by a 3 character short hexadecimal color, which means the 768 char string is the exact size needed to store 256 pixels, starting from left to right.  As an example, let's look at storing the following image on PixelMap.io.

![Star Tile](https://raw.githubusercontent.com/Pixel-Map/pixelmap.io/master/images/star.png "Star Tile Example")

This tile is stored as the following one-line string:
```
390390390390390390390000000390390390390390390390390390390390390390000FF0FF0000390390390390390390390390390390390390000FF0FF0000390390390390390390390390390390390000FF0FF0FF0FF0000390390390390390000000000000000000FF0FF0FF0FF0000000000000000000000FF0FF0FF0FF0FF0FF0FF0FF0FF0FF0FF0FF0FF0FF0000390000FF0FF0FF0FF0000FF0FF0000FF0FF0FF0FF0000390390390000FF0FF0FF0000FF0FF0000FF0FF0FF0000390390390390390000FF0FF0000FF0FF0000FF0FF0000390390390390390390000FF0FF0FF0FF0FF0FF0FF0FF0000390390390390390000FF0FF0FF0FF0FF0FF0FF0FF0FF0FF0000390390390390000FF0FF0FF0FF0FF0FF0FF0FF0FF0FF0000390390390000FF0FF0FF0FF0FF0000000FF0FF0FF0FF0FF0000390390000FF0FF0FF0000000390390000000FF0FF0FF0000390000FF0FF0000000390390390390390390000000FF0FF0000000000000390390390390390390390390390390000000000
```

However, if you break the string apart into 16 lines, it looks like the following:
![StarHex](https://raw.githubusercontent.com/Pixel-Map/pixelmap.io/master/images/starhex.png "Starhex")

The 3 digit color codes can be easily referenced here: [https://en.wikipedia.org/wiki/Web_colors#Web-safe_colors](https://en.wikipedia.org/wiki/Web_colors#Web-safe_colors)
On this image, the following colors were used:

![Colors](https://raw.githubusercontent.com/Pixel-Map/pixelmap.io/master/images/colors.png "Star Used Colors")

We're currently working on a tile editor that should make it MUCH easier to
generate the hexadecimal string, but it's currently a WIP.

Once a pixel is updated, the website should update within about 5-10 minutes.  If for any reason it doesn't work, feel free to contact me at ken@devopslibrary.com.
```
pragma solidity ^0.4.2;
contract PixelMap {
    mapping (uint => address) public owners;
    mapping (uint => string) public images;
    mapping (uint => string) public urls;
    mapping (uint => uint) public prices;
    address creator;
    event TileUpdated(uint x, uint y, string image, string url, uint price, address owner);
    event OwnerUpdated(uint x, uint y, address owner);

    // Constructor
    function PixelMap() {
        creator = msg.sender;
    }
    // Given X & Y, return Tile number.
    function getPos(uint x, uint y) returns (uint) {
        return y*81+x;
    }

    // Get Tile information at X,Y position.
    function getTile(uint x, uint y) returns (address, string, string) {
        uint location = getPos(x, y);
        return (owners[location], urls[location], images[location]);
    }

    // Purchase an unclaimed Tile for 10 Eth.
    function buyTile(uint x, uint y) payable {
        uint location = getPos(x, y);
        uint price = 2000000000000000000;
        if (owners[location] == msg.sender) {
            throw; // You already own this pixel silly!
        }
        // If Unowned by the Bank
        if (owners[location] == 0x0) {
            if (msg.value == 2000000000000000000) {
                // Send to Creator
                if (creator.send(2000000000000000000)) {
                    owners[location] = msg.sender;
                    prices[location] = 0; // Set Price to 0.  0 is not for sale.
                    OwnerUpdated(x, y, msg.sender);
                }
                else {throw;}
            }
            else {
                throw; // 10 Eth not supplied
            }
        }
        else {
            if (owners[location] != 0x0) {
                price = prices[location];
                if (price == 0) {throw;} // Tile not for sale!
                else {
                    if (msg.value == price) {
                        if (owners[location].send(price)) {
                            // Set New Owner
                            owners[location] = msg.sender;
                            prices[location] = 0; // Set Price to 0.
                            OwnerUpdated(x, y, msg.sender);
                        }
                        else {throw;}
                    }
                }
            }
        }
    }

    // Set an already owned Tile to whatever you'd like.
    function setTile(uint x, uint y, string image, string url, uint price) {
        uint location = getPos(x, y);
        if (owners[location] != msg.sender) {throw;} // Pixel not owned by you!
        if (bytes(image).length != 768) {throw;} // Incorrect String Length Provided!
        else {
            images[location] = image;
            urls[location] = url;
            prices[location] = price;
            TileUpdated(x, y, image, url, price, msg.sender);
        }
    }
}
```
