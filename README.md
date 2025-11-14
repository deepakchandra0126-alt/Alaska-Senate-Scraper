KYC2020 Assignment – Alaska Senate Scraper 
Name: [Deepak Chandra Papnoi]   
Roll No: [07976803122]   
Branch: B.Tech – [IT]   
Batch: 2026   
# Technologies Used- Java 17   - Selenium WebDriver (Headless Chrome)   - Jsoup (HTML Parsing)   - Jackson (JSON Output)   
## Time Taken: 3 Hours 10 Minutes - Page Analysis: 35 mins   - Code Development: 80 mins   - Email Decoding (Cloudflare): 30 mins   - Testing & JSON Validation: 30 mins   - Documentation: 15 mins   --- 
## Java Code (ScrapeSenate.java) 
```java 
import org.openqa.selenium.*; 
import org.openqa.selenium.chrome.*; 
import org.jsoup.*; 
import org.jsoup.nodes.*; 
import org.jsoup.select.*; 
import com.fasterxml.jackson.databind.*; 
import java.io.*; 
import java.util.*; 
 
public class ScrapeSenate { 
    public static void main(String[] args) { 
        WebDriver driver = null; 
        try { 
            ChromeOptions options = new ChromeOptions(); 
            options.addArguments("--headless", "--no-sandbox", "--disable-dev-shm-usage"); 
            driver = new ChromeDriver(options); 
            driver.get("https://akleg.gov/senate.php"); 
 
            Document doc = Jsoup.parse(driver.getPageSource()); 
            Elements items = doc.select("ul li"); 
 
            List<Map<String, String>> data = new ArrayList<>(); 
            JavascriptExecutor js = (JavascriptExecutor) driver; 
 
            for (Element li : items) { 
                Elements links = li.select("a[href*='Member/Detail']"); 
                if (links.isEmpty()) continue; 
 
                Map<String, String> s = new HashMap<>(); 
                Element a = links.first(); 
                s.put("Name", a.text().trim()); 
                s.put("URL", a.attr("abs:href")); 
                s.put("Title", "Senator"); 
 
                String text = li.text(); 
                s.put("Position", getValue(text, "District:")); 
                s.put("Party", getValue(text, "Party:")); 
                s.put("Address", getValue(text, "City:")); 
                s.put("Phone", getValue(text, "Phone:")); 
 
                Elements emailLink = li.select("a[href*='email-protection']"); 
                if (!emailLink.isEmpty()) { 
                    try { 
                        WebElement el = driver.findElement(By.cssSelector("a[href*='email-protection']")); 
                        String href = el.getAttribute("href"); 
                        String script = "return (function(){var a='" + href.split("#")[1] + "';var b='';for(var 
i=0;i<a.length;i+=2){b+=String.fromCharCode(parseInt(a.substr(i,2),16))}return b})();"; 
                        String email = (String) js.executeScript(script); 
                        s.put("Email", email); 
                    } catch (Exception e) { 
                        s.put("Email", "N/A"); 
                    } 
                } 
 
                data.add(s); 
            } 
 
            new ObjectMapper().writerWithDefaultPrettyPrinter().writeValue(new 
File("senate_data.json"), data); 
            System.out.println("Done! " + data.size() + " senators saved."); 
 
        } catch (Exception e) { 
            e.printStackTrace(); 
        } finally { 
            if (driver != null) driver.quit(); 
        } 
    } 
 
    private static String getValue(String text, String prefix) { 
        for (String line : text.split("\n")) { 
            if (line.trim().startsWith(prefix)) { 
                return line.split(":", 2)[1].trim(); 
            } 
        } 
        return "N/A"; 
    } 
} 
