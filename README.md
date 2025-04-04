package ma.sg.its.rpaportal.services.impl;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import ma.sg.its.rpaportal.services.OrchestratorService;
import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVParser;
import org.apache.poi.openxml4j.exceptions.InvalidFormatException;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.springframework.boot.autoconfigure.condition.ConditionalOnExpression;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.io.File;
import java.io.FileReader;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import static ma.sg.its.rpaportal.services.impl.ProcessCCServiceImpl.*;

@RequiredArgsConstructor
@Slf4j
@Component
@ConditionalOnExpression("${rpa-portal.seed.orchestrator.enabled}")
public class FileWatcher {
    private final OrchestratorService service;
    public List<String> directories = Arrays.asList(
            "/data/RPA/ATD/input",
            "/data/RPA/CC/input",
            "/data/RPA/DDR/input",
            "/data/RPA/RES/input",
            "/data/RPA/SA/input",
            "/data/RPA/SCC/input"
    );

    @Scheduled(cron = "${rpa-portal.seed.orchestrator.cron}")
    public void checkInputDirectoriesForNewFiles() throws IOException, InvalidFormatException {
        log.info("[FileWatcher - checkInputDirectoriesForNewFiles] start");
        for (String directory : directories) {
            File[] files = new File(directory).listFiles();
            if (files != null) {
                for (File file : files) {
                    if(file.isFile()){
                        log.info("[FileWatcher - checkInputDirectoriesForNewFiles] Found A new File: {}", file.getAbsolutePath());
                        Workbook workbook ;
                        int fileLenght=0;
                        Sheet sheet;
                        if(file.getParentFile().getParentFile().getName().equals("ATD")) {
                            try (XSSFWorkbook workbookResource = new XSSFWorkbook(file)) {
                                workbook = workbookResource;
                                sheet = workbook.getSheet("Nouvelle ATD");
                                fileLenght = sheet.getLastRowNum();
                            }
                             catch (IOException e) {
                                    e.printStackTrace(); // Handle exception appropriately
                                    log.error("Error processing workbook for file {}: {}", file.getAbsolutePath(), e.getMessage());
                                }
                        }
                        else if(file.getParentFile().getParentFile().getName().equals("CC"))
                        {
                            try(FileReader fileReader = new FileReader(file.getAbsolutePath())){
                                CSVParser parser = new CSVParser(fileReader, CSVFormat.RFC4180
                                        .withHeader(this.headersCC())
                                        .withIgnoreEmptyLines()
                                        .withDelimiter(';')
                                        .withFirstRecordAsHeader());
                                fileLenght=parser.getRecords().size();
                            }
                            catch (IOException e) {
                                e.printStackTrace(); // Handle exception appropriately
                                log.error("Error processing CSV for file {}: {}", file.getAbsolutePath(), e.getMessage());
                            }
                        }
                        else
                        {
                            workbook = new XSSFWorkbook(file);
                            sheet = workbook.getSheetAt(0);
                            fileLenght = sheet.getLastRowNum();
                        }
                        boolean dicision = service.takeDecision(fileLenght, file.getParentFile().getParentFile().getName());
                        if(dicision)
                            moveFile(file.getAbsolutePath(), file.getParentFile().getParentFile() + "/archive");
                    }
                    else{
                        log.info("[FileWatcher - checkInputDirectoriesForNewFiles] not found");
                    }
                    }
            }
        }
        log.info("[FileWatcher - checkInputDirectoriesForNewFiles] End");
    }
    public String[] headersCC() {
        return new String[]{TYPE_MESSAGE, MODULE, BIC_DONNEUR_ORDER, NUM_CHRONO, MESSAGE_20, STATUT, DATE_INTEGRATION,
                DATE_VALEUR, CODE_BIC_BANQUE, RIB_BENEFICIAIRE, NOM_BENEFICIAIRE, MESSAGE_52A, NATURE_FRAIS, MONTANT_SANS_FRAIS, MOTIF_OPERATION};
    }

    public void moveFile(String filePath, String target) throws IOException {
        Path sourcePath = Paths.get(filePath);
        Path targetPath = Paths.get(target);

        try {
            // Move the file
            Files.move(sourcePath, targetPath.resolve(sourcePath.getFileName()), StandardCopyOption.REPLACE_EXISTING);

            log.info("File with path {} moved successfully to {}", filePath, target);
        } catch (IOException e) {
            log.error("Error moving the file {} to {}", filePath, target, e);
            throw e; // Rethrow the IOException
        }
    }

}
   <dependency>
            <!-- this is needed or IntelliJ gives junit.jar or junit-platform-launcher:1.3.2 not found errors -->
            <groupId>org.junit.platform</groupId>
            <artifactId>junit-platform-launcher</artifactId>
        </dependency>
        
