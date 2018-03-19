---
layout: post
title: mysql的urlencode的存储过程
description: 由于mysql不支持urlencode,只能增加一个sp
category: blog
---

```

CREATE FUNCTION urlencode(str VARCHAR(4096) CHARSET utf8) RETURNSVARCHAR(4096) CHARSET utf8
DETERMINISTIC
CONTAIN SSQL
BEGIN
  -- the individual character we are converting in our loop
-- NOTE: must be VARCHAR even though it won't vary in length
  -- CHAR(1), when used with SUBSTRING, made spaces '' instead of ' '
  DECLARE sub VARCHAR(1) CHARSET utf8;
  -- the ordinal value of the character (i.e. ? becomes 50097)
  DECLARE val BIGINTDEFAULT0;
  -- the substring index we use in our loop (one-based)
  DECLARE ind INTDEFAULT1;
  -- the integer value of the individual octet of a character being encoded
  -- (which is potentially multi-byte and must be encoded one byte at a time)
  DECLARE oct INTDEFAULT0;
  -- the encoded return string that we build up during execution
  DECLARE ret VARCHAR(4096)DEFAULT'';
  -- our loop index for looping through each octet while encoding
  DECLARE octind INT DEFAULT 0;
  IF ISNULL(str) THEN
      RETURN NULL;
  ELSE
  SET ret = '';
      -- loop through the input string one character at a time - regardless
      -- of how many bytes a character consists of
      WHILE ind <= CHAR_LENGTH(str) DO
        SET sub = MID(str, ind, 1);
        SET val = ORD(sub);
        -- these values are ones that should not be converted
        -- see http://tools.ietf.org/html/rfc3986
        IF NOT (val BETWEEN 48 AND 57 OR    -- 48-57  = 0-9
                val BETWEEN 65 AND 90 OR    -- 65-90  = A-Z
                val BETWEEN 97 AND 122 OR    -- 97-122 = a-z
                -- 45 = hyphen, 46 = period, 95 = underscore, 126 = tilde
                val IN (45, 46, 95, 126)) THEN
            -- This is not an "unreserved" char and must be encoded:
            -- loop through each octet of the potentially multi-octet character
            -- and convert each into its hexadecimal value
            -- we start with the high octect because that is the order that ORD
            -- returns them in - they need to be encoded with the most significant
            -- byte first
            SET octind = OCTET_LENGTH(sub);
            WHILE octind > 0 DO
              -- get the actual value of this octet by shifting it to the right
              -- so that it is at the lowest byte position - in other words, make
              -- the octet/byte we are working on the entire number (or in even
              -- other words, oct will no be between zero and 255 inclusive)
              SET oct = (val >> (8 * (octind - 1)));
              -- we append this to our return string with a percent sign, and then
              -- a left-zero-padded (to two characters) string of the hexadecimal
              -- value of this octet)
              SET ret = CONCAT(ret, '%', LPAD(HEX(oct), 2, 0));
              -- now we need to reset val to essentially zero out the octet that we
              -- just encoded so that our number decreases and we are only left with
              -- the lower octets as part of our integer
              SET val = (val & (POWER(256, (octind - 1)) - 1));
              SET octind = (octind - 1);
            END WHILE;
        ELSE
            -- this character was not one that needed to be encoded and can simply be
            -- added to our return string as-is
          SET ret = CONCAT(ret, sub);
        END IF;
        SET ind = (ind + 1);
    END WHILE;
  END IF;
  RETURN ret;
END;
```

## 最后
我的微博[van1988ch](https://weibo.com/2296015293/profile)