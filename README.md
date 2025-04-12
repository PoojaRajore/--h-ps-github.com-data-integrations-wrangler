fragment DIGIT : [0-9];
fragment LETTER : [a-zA-Z];

BYTE_UNIT : ('B' | 'KB' | 'MB' | 'GB' | 'TB');
TIME_UNIT : ('ms' | 's' | 'm' | 'h');

BYTE_SIZE : DIGIT+ ('.' DIGIT+)? BYTE_UNIT;
TIME_DURATION : DIGIT+ ('.' DIGIT+)? TIME_UNIT;
