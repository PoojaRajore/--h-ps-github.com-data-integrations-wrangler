fragment DIGIT : [0-9];
fragment LETTER : [a-zA-Z];

BYTE_UNIT : ('B' | 'KB' | 'MB' | 'GB' | 'TB');
TIME_UNIT : ('ms' | 's' | 'm' | 'h');

BYTE_SIZE : DIGIT+ ('.' DIGIT+)? BYTE_UNIT;
TIME_DURATION : DIGIT+ ('.' DIGIT+)? TIME_UNIT;
public class ByteSize extends Token {
    private final double size;
    private final String unit;

    public ByteSize(String value) {
        super("BYTE_SIZE", value);
        this.unit = value.replaceAll("[\\d.]", "");
        this.size = Double.parseDouble(value.replaceAll("[^\\d.]", ""));
    }

    public long getBytes() {
        switch (unit.toUpperCase()) {
            case "B": return (long) size;
            case "KB": return (long) (size * 1024);
            case "MB": return (long) (size * 1024 * 1024);
            case "GB": return (long) (size * 1024 * 1024 * 1024);
            case "TB": return (long) (size * 1024L * 1024 * 1024 * 1024);
            default: throw new IllegalArgumentException("Unknown unit: " + unit);
        }
    }
}
public class ByteSize extends Token {
    private final double size;
    private final String unit;

    public ByteSize(String value) {
        super("BYTE_SIZE", value);
        this.unit = value.replaceAll("[\\d.]", "");
        this.size = Double.parseDouble(value.replaceAll("[^\\d.]", ""));
    }

    public long getBytes() {
        switch (unit.toUpperCase()) {
            case "B": return (long) size;
            case "KB": return (long) (size * 1024);
            case "MB": return (long) (size * 1024 * 1024);
            case "GB": return (long) (size * 1024 * 1024 * 1024);
            case "TB": return (long) (size * 1024L * 1024 * 1024 * 1024);
            default: throw new IllegalArgumentException("Unknown unit: " + unit);
        }
    }
}
