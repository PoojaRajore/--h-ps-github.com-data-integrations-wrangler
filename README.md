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
public class TimeDuration extends Token {
    private final double duration;
    private final String unit;

    public TimeDuration(String value) {
        super("TIME_DURATION", value);
        this.unit = value.replaceAll("[\\d.]", "");
        this.duration = Double.parseDouble(value.replaceAll("[^\\d.]", ""));
    }

    public long getMilliseconds() {
        switch (unit) {
            case "ms": return (long) duration;
            case "s": return (long) (duration * 1000);
            case "m": return (long) (duration * 60 * 1000);
            case "h": return (long) (duration * 60 * 60 * 1000);
            default: throw new IllegalArgumentException("Unknown unit: " + unit);
        }
    }
}
@Override
public Token visitByteSizeValue(DirectivesParser.ByteSizeValueContext ctx) {
    return new ByteSize(ctx.getText());
}

@Override
public Token visitTimeDurationValue(DirectivesParser.TimeDurationValueContext ctx) {
    return new TimeDuration(ctx.getText());
}
@Directive(
  name = "aggregate-stats",
  description = "Aggregates byte size and time duration columns"
)
public class AggregateStats implements Directive {
    private String byteColumn;
    private String timeColumn;
    private String outputByteColumn;
    private String outputTimeColumn;

    private long totalBytes = 0;
    private long totalMillis = 0;
    private int count = 0;

    @Override
    public UsageDefinition define() {
        return UsageDefinition.builder()
            .define("byteColumn", TokenType.COLUMN_NAME)
            .define("timeColumn", TokenType.COLUMN_NAME)
            .define("outputByteColumn", TokenType.COLUMN_NAME)
            .define("outputTimeColumn", TokenType.COLUMN_NAME)
            .build();
    }

    @Override
    public void initialize(Arguments args) {
        byteColumn = ((ColumnName) args.value("byteColumn")).value();
        timeColumn = ((ColumnName) args.value("timeColumn")).value();
        outputByteColumn = ((ColumnName) args.value("outputByteColumn")).value();
        outputTimeColumn = ((ColumnName) args.value("outputTimeColumn")).value();
    }

    @Override
    public List<Row> execute(List<Row> rows, ExecutorContext context) {
        for (Row row : rows) {
            ByteSize size = new ByteSize(row.getValue(byteColumn).toString());
            TimeDuration time = new TimeDuration(row.getValue(timeColumn).toString());

            totalBytes += size.getBytes();
            totalMillis += time.getMilliseconds();
            count++;
        }

        Row result = new Row();
        result.add(outputByteColumn, totalBytes / (1024.0 * 1024.0)); // Convert to MB
        result.add(outputTimeColumn, totalMillis / 1000.0); // Convert to seconds

        return Collections.singletonList(result);
    }

    @Override
    public void destroy() { }
}
@Test
public void testByteSizeParsing() {
    ByteSize size = new ByteSize("2.5MB");
    Assert.assertEquals(2.5 * 1024 * 1024, size.getBytes(), 0.01);
}
@Test
public void testAggregateStatsDirective() throws Exception {
    List<Row> rows = Arrays.asList(
        new Row().add("data_transfer_size", "1MB").add("response_time", "500ms"),
        new Row().add("data_transfer_size", "2MB").add("response_time", "1500ms")
    );

    String[] recipe = new String[] {
        "aggregate-stats :data_transfer_size :response_time :total_size_mb :total_time_sec"
    };

    List<Row> result = TestingRig.execute(recipe, rows);
    Assert.assertEquals(1, result.size());
    Assert.assertEquals(3.0, result.get(0).getValue("total_size_mb"), 0.001);
    Assert.assertEquals(2.0, result.get(0).getValue("total_time_sec"), 0.001);
}
### ByteSize and TimeDuration Parsers

Wrangler now supports native parsing of size and duration units:

- **ByteSize** supports: `B`, `KB`, `MB`, `GB`, `TB`
- **TimeDuration** supports: `ms`, `s`, `m`, `h`

**Example Usage:**
```wrangler
aggregate-stats :data_transfer_size :response_time :total_size_mb :total_time_sec
