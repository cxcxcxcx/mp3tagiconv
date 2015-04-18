# mp3tagiconv
Automatically exported from code.google.com/p/mp3tagiconv

mp3tagiconv是一款可用于解决MP3乱码问题的软件。它可以自动探测MP3标签的编码，并以合理的方式重新写入，经过其处理的编码可被Linux、Windows（包括Windows Media Player）和各种移动设备正确识别。（原理见[这里](http://linux-wiki.cn/wiki/Mp3%E6%A0%87%E7%AD%BE%E4%B9%B1%E7%A0%81%E9%97%AE%E9%A2%98%E5%88%86%E6%9E%90%E4%B8%8E%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88)）：

mp3tagconv is a Mutagen based script intending to convert the encoding of tags of MP3 files, so that it can be recognized correctly by music players in Linux, Windows, etc.

The program is now only tested in Simplified Chinese, we will be happy if information about other languages is provided.

If you want to know more about how it works, please refer to [An explanation in Chinese](http://linux-wiki.cn/wiki/Mp3%E6%A0%87%E7%AD%BE%E4%B9%B1%E7%A0%81%E9%97%AE%E9%A2%98%E5%88%86%E6%9E%90%E4%B8%8E%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88).

## Usage ##
**The script depends on `mutagen` to run.** Please install it from the repository of your distribution (in Ubuntu: `python-mutagen`).

For mp3 files with Chinese tags(we first try gbk, then utf8), ID3v2 tags which are already encoded in unicode will not be affected:
> `mp3tagiconv a.mp3`

You can use -e to specify the encoding used if the tag is stored by latin-1. The program will guess your encoding according to your list:
> `mp3tagiconv -e gbk,utf8 b.mp3`

If you don't want to confirm for every file(not recommended):
> `mp3tagiconv --do-update *.mp3`


## Details ##
Actually, the program does the following things:
  * Read tags of a MP3 file from its ID3v2 and ID3v1. ID3v1 is used to supplement the result of ID3v2.
  * Guess the encoding of the tags.
  * Decode the tags, then:
    * Write directly to ID3v2 tag of MP3 file via mutagen.
    * Encode the tags with local encoding(eg. gbk in Chinese) and write it to ID3v1 tag of the file.
  * The modified tag now can be recognized by most of the applications, eg. rthymbox, totem, juk, Windows Media Player, etc.
