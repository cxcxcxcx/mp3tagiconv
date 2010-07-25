#!/usr/bin/env python
# Reencode the ID3v1 and ID3v2 tag of a mp3 file.
# Copyright 2010 CHEN, Xing (cxcxcxcx@gmail.com)
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of version 2 of the GNU General Public License as
# published by the Free Software Foundation.
#

import os
import sys
import locale

from optparse import OptionParser
import mutagen.id3

VERSION = (0, 1)

def isascii(string):
    return not string or min(string) < '\x127'

def MakeID3v1(id3, enc):
    """Return an ID3v1.1 tag string from a dict of ID3v2.4 frames."""

    """Copied from id3.py of mutagen. What I modified is the encoding of ID3v1, so that the tag can be parsed
    correctly by Windows Media Player, etc."""

    v1 = {}

    for v2id, name in {"TIT2": "title", "TPE1": "artist",
                       "TALB": "album"}.items():
        if v2id in id3:
            text = id3[v2id].text[0].encode(enc, 'replace')[:30]
        else:
            text = ""
        v1[name] = text + ("\x00" * (30 - len(text)))

    if "COMM" in id3:
        cmnt = id3["COMM"].text[0].encode(enc, 'replace')[:28]
    else: cmnt = ""
    v1["comment"] = cmnt + ("\x00" * (29 - len(cmnt)))

    if "TRCK" in id3:
        try: v1["track"] = chr(+id3["TRCK"])
        except ValueError: v1["track"] = "\x00"
    else: v1["track"] = "\x00"

    if "TCON" in id3:
        try: genre = id3["TCON"].genres[0]
        except IndexError: pass
        else:
            if genre in TCON.GENRES:
                v1["genre"] = chr(TCON.GENRES.index(genre))
    if "genre" not in v1: v1["genre"] = "\xff"

    if "TDRC" in id3:
        v1["year"] = str(id3["TDRC"])[:4]
    elif "TYER" in id3:
        v1["year"] = str(id3["TYER"])[:4]
    else:
        v1["year"] = "\x00\x00\x00\x00"

    return ("TAG%(title)s%(artist)s%(album)s%(year)s%(comment)s"
            "%(track)s%(genre)s") % v1 

def has_id3v1(filename):
    f = open(filename, 'rb+')
    try: f.seek(-128, 2)
    except IOError: pass
    else: return (f.read(3) == "TAG")

def get_id3v1(filename):
    id3v1 = None
    f = open(filename, 'rb+')
    try: f.seek(-128, 2)
    except IOError: pass
    else: 
        id3v1 = mutagen.id3.ParseID3v1(f.read(128))
    return id3v1

def convTxt(encList, s):
    decF = s
    for i in encList:
        try:
            decF = s.encode('latin1').decode(i)
            tagEnc = i
            break
        except:
            pass
    #print tagEnc
    return decF

def saveTagToFile(filename, fileID3, v1enc):
    """ Update the file."""
    fileID3.save(v1=1)

    try:
        hasID3v1 = has_id3v1(filename)
        try: f = open(filename, 'rb+')
        except IOError, err:
            from errno import ENOENT
            if err.errno != ENOENT: raise
            f = open(filename, 'ab') # create, then reopen
            f = open(filename, 'rb+')

        v1 = 2
        if hasID3v1:
            f.seek(-128, 2)
            if v1 > 0: f.write(MakeID3v1(fileID3, v1enc))
            else: f.truncate()
        elif v1 == 2:
            f.seek(0, 2)
            f.write(MakeID3v1(fileID3, v1enc))
    finally:
        f.close()

def main(argv):
    from mutagen import File

    default_enclist = "utf8,gbk"

    mutagen_version = ".".join(map(str, mutagen.version))
    my_version = ".".join(map(str, VERSION))
    version = "mp3tagiconv %s\nUses Mutagen %s" % (
        my_version, mutagen_version)

    parser = OptionParser(usage="%prog [OPTION] [FILE]...", version=version,
            description=("Mutagen-based mp3 tag encoding converter, which "
                            "can automatically detect the encoding "
                        "of the tags of a MP3 file, and can convert it so "
                        "that the tags can be recognized correctly in most "
                        "applications."))
    parser.add_option(
        "-e", "--encoding-list", metavar="ENCODING_LIST", action="store",
        type="string", dest="enclist",
        help=("Specify original tag encoding, comma seperated if you want "
            "me to guess one by one(default: %s)" % default_enclist))
    parser.add_option(
        "--v1-encoding", metavar="ID3V1_ENCODING", action="store",
        type="string", dest="v1enc",
        help=("Specify tag encoding for ID3v1, the default value(gbk) should **ALWAYS** be OK for Simplified Chinese tags."))
    parser.add_option(
        "--do-update", action="store_true", dest="notest",
        help="Save updates WITHOUT confirming. It is NOT recommended.")
    #parser.add_option(
        #"--confirm-test", action="store_true", dest="notest",
        #help="Actually modify files. This is NOT default in order to protect your mp3 file from being damaged by wrong parameters.")

    (options, args) = parser.parse_args(argv[1:])
    if not args:
        raise SystemExit(parser.print_help() or 1)

    enclist = options.enclist or default_enclist
    enclist = enclist.split(',')

    notest = options.notest
    v1enc = options.v1enc or "gbk"

    enc = locale.getpreferredencoding()
    for filename in args:
        print "--", filename
        try: 
            # Prepare information from ID3v1
            id3v1 = get_id3v1(filename)

            fileID3 = mutagen.id3.ID3(filename)
            if id3v1 is not None:
                # Merge ID3v1 and ID3v2
                for i in id3v1.keys(): 
                    if i not in fileID3.keys():
                        fileID3.add(id3v1[i])

            #print f.pprint()
            for tag in filter(lambda t: t.startswith("T"), fileID3):
                frame = fileID3[tag]
                if isinstance(frame, mutagen.id3.TimeStampTextFrame):
                    # non-unicode fields
                    continue

                try:
                    text = frame.text
                except AttributeError:
                    continue

                try:
                    #print frame.encoding
                    if frame.encoding == 0:
                        text = map(convTxt, [enclist,]*len(frame.text), frame.text)
                except (UnicodeError, LookupError):
                    continue
                else:
                    frame.text = text
                    #for i in text:
                        #print i.encode('utf8')
                    if not text or min(map(isascii, text)):
                        frame.encoding = 3
                    else:
                        frame.encoding = 1
                print frame.pprint().encode(enc)

            doSave = False
            if notest:
                doSave = True
            else:
                #print fileID3.pprint().encode(enc)
                print "Do you want to save?(y/N)",
                p=raw_input()
                doSave = p=="y" or p=="Y"

            if doSave:
                print "*** Saving..."
                saveTagToFile(filename, fileID3, v1enc)
            else:
                print
                print "*** File is NOT updated."

        except AttributeError: print "- Unknown file type" 
        except KeyboardInterrupt: raise
        except Exception, err: print str(err)
        print

if __name__ == "__main__":
    try: import mutagen
    except ImportError:
        sys.path.append(os.path.abspath("../"))
        import mutagen
    main(sys.argv)
