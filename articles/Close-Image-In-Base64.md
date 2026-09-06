---
title: Close Image in Base64
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Close Image in Base64

Today I'm futzing around with localizing images that are in a ImageList object (which currently doesn't seem as easy as an endeavor as I thought it would be). As I was perusing the .resx file, I found another Base64-encoded image:

```
        Qk2qBAAAAAAAADYAAAAoAAAAEwAAABMAAAABABgAAAAAAAAAAADEDgAAxA4AAAAAAAAAAAAAKES1KES1
        DSumETCsEjGrFDKpEjCpEzKsEzKtEDGuDTCxCi6yCC20BiqxAyixASe0ACSuHzysDzGyLZb/KEnNEzjM
        Gz7MIUTPJUfPJkfPJEbPI0fQIUbRHkXSG0XVFkLVE0DWDj3VCTnWBDXVATDRACrCHzyrLZb/DjfYHEPb
        KEzdL1LeM1XfNFfgMVXgMVbgLlbhKlXiJVPkH1DkGUzlE0jlDUPlBz7kAjjgAC/RACOqLZb/FDzdJUvg
        MVThOVvjPV7kPV/kO17kOV7lN17lMl3mLFvnJVfoH1TpGVDpEUrpC0ToBT3kAjPWASavLZb/GEDeLlHh
        OlzjQmLktcH1////Q2XmQWXmPWTmOGLnMmDoKlzpJFnq////obj3DkjpCUDlBTbXAiewLZb/HkXfNlji
        QmLkSWfl////////////RWjmQWfnO2TnNWLoLV7q////////////EUrpDUTmCzvXBiuxLZb/JUvgPV7j
        SGflTmzmUG3m////////////QmjnPGXnNmLo////////////GVDpFkzoE0fkED/YCi2xLZb/K0/hRGPk
        TmvmUm/nU2/nUW7m////////////PWXn////////8fX+IVToG1DoGE3nGErkFkLXDzGyLZb/MlTiS2nl
        VHDnVnHnVXDoUm/nTWzm////////////////////JlXnIVLnHE7mHE3mHEzjHEbWEjOwLZb/N1njUW3n
        V3LoWXPoV3HnUm7mTWrmR2fl////////////LFflJlPlIlDlHk3lH03lIU7iIUnWFjWvLZb/PF3jWXTo
        XXfoXXboWXLnU2/mTmvl////////////////////J1HkI0/kIEzjIk3jJU/hJUrVGTevLZb/RmXlYHro
        YnzoYHroW3ToVG7n////////////Olzj////////////JE3iIkviJU3iKVDgKUzUGzmuLZb/RmXlaIHp
        aoPqZX7pXnfo////////////Q2HkPV3kNljj////////////J07iKlDiLFHgKk3THDmtLZb/TWrmbofr
        cIrraoPq////////////TWjlR2TkQF/kOlvjNFbi////////////LVLiLlLfLE3THTqtLZb/Um/neZHs
        fJPscYrrwsz3////WXLnU23nTWjlRmTkQV/kOlvjOFnj////rbvzMVXiMFPfLE7THTqtLZb/WHToiZ3u
        jaHvf5XtcYrraoPqZH3pXnjoWXPnVG/nUW3mS2jlSmjlRGPkP2DkOlzjNFbgKkzSGDasLZb/ZX/pl6nw
        mqzxiZ3uepHscovrbYXqaIHpZX7oYnzpYXvpXXfoWXToUm/mTGrlQ2PkN1nhKErSFTOqLZb/do3skKPv
        lKbwhZrudY3rbofqaIHqZn/pYnzpX3noXnnoWXXoVnLnUG7mSWjlP2DkMlXgI0bQMEu2LZb/ZX/pc4vr
        YHvpVXHnTWrmSGblQmHkRGPkPl/kPl/kO1zjO1zjM1biM1biMFPiKE3gH0XcNFPQDzGyLZb/
```

The funny thing is that blob of text is actually this image:

![Close Image](https://jasonbock.net/images/CloseImage.png "Close Image")

Coincidence? I think not!

> Published: 12.20.2004 03:44:02 PM CST