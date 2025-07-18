import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

String input = "18/07/2025"; // dd/MM/yyyy

DateTimeFormatter inputFormatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
DateTimeFormatter dbFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");

LocalDate date = LocalDate.parse(input, inputFormatter);
String formattedDate = date.format(dbFormatter); // "2025-07-18"

crsRequestTrack.setRtDate(formattedDate);