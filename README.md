Thanks! You’ve shared all the key pieces:

Java code that saves a new CrsRequestTrack

Oracle table schema

Your entity class


Now let’s pinpoint why you're getting:

> ❌ DataIntegrityViolationException




---

🔍 Root Cause: Date Fields as Strings

Your entity has 3 DATE columns in Oracle:

"RT_DATE"       DATE DEFAULT sysdate,
"RT_ACTION_DT"  DATE,
"RT_QED"        DATE NOT NULL ENABLE,

But in your Java entity, they are all defined as:

public String rtDate;
public String rtActionDt;
public String rtQed;

You're assigning this in your Java code:

crsRequestTrack.setRtQed(quarterEndDate); // quarterEndDate is likely a String in dd/MM/yyyy

If quarterEndDate is "18/07/2025" → this will fail, because Oracle expects a valid DATE, but you're passing a String that cannot be parsed correctly by Hibernate.


---

✅ FIX

✅ Step 1: Convert String to LocalDate

Change the date fields in the entity from String to LocalDate (or java.util.Date if you're not using Java 8+):

@Column(name="RT_QED")
private LocalDate rtQed;

@Column(name="RT_DATE", insertable = false, updatable = false)
private LocalDate rtDate; // handled by Oracle default sysdate

@Column(name="RT_ACTION_DT")
private LocalDate rtActionDt;

> ☝️ RT_DATE has a DEFAULT sysdate — so you don’t need to set it manually.



✅ Step 2: Format the input

In your code, parse the date from dd/MM/yyyy:

DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
LocalDate qedDate = LocalDate.parse(quarterEndDate, formatter);
crsRequestTrack.setRtQed(qedDate);

✅ Step 3: Do not set RT_DATE manually

Let Oracle handle it with DEFAULT sysdate, so remove any call like:

crsRequestTrack.setRtDate(...); // ❌ Don’t do this


---

✅ Updated Entity (Partial)

@Entity
@Table(name = "CRS_REQUEST_TRACK")
public class CrsRequestTrack {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "crs_seq")
    @SequenceGenerator(name = "crs_seq", sequenceName = "CRS_REQUEST_TRACK_SEQ", allocationSize = 1)
    @Column(name="RT_ID")
    private Long rtId;

    @Column(name="RT_MAKER")
    private String rtMaker;

    @Column(name="RT_DATE", insertable = false, updatable = false)
    private LocalDate rtDate;

    @Column(name="RT_STATUS")
    private Integer rtStatus;

    @Column(name="RT_TYPE")
    private String rtType;

    @Column(name="RT_SUBTYPE")
    private String rtSubType;

    @Column(name="RT_CHECKER")
    private String rtChecker;

    @Column(name="RT_ACTION_DT")
    private LocalDate rtActionDt;

    @Column(name="RT_QED")
    private LocalDate rtQed;

    @Column(name="RT_BRANCH")
    private String rtBranch;

    @Column(name="RT_FILE_NAME")
    private String rtFileName;
}


---

✅ Summary

Problem	Fix

RT_QED is NOT NULL in DB, but you're passing string (invalid format)	Change to LocalDate in entity, and parse input date
RT_DATE has default in DB	Let Oracle handle it; do not set it manually
Entity has String instead of Date types	Replace with LocalDate or Date



---

Let me know if you want the java.util.Date version or complete working code based on your quarterEndDate variable.

