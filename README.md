public class FileWatcher {
    private final OrchestratorService service;
    public List<String> directories = Arrays.asList(
Make directories a static final constant or non-public and provide accessors if needed.
Why is this an issue?

7 days ago
L34
Code Smell
Minor

Open

BAHBAH Mustapha
10min effort

Comment

cwe
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
                        if(file.getParentFile().getParentFile().getName().equals("ATD"))
                        {
                            workbook = new XSSFWorkbook(file);
                            sheet = workbook.getSheet("Nouvelle ATD");
                            fileLenght = sheet.getLastRowNum();
                        }
                        else if(file.getParentFile().getParentFile().getName().equals("CC"))
                        {
                            FileReader fileReader = new FileReader(file.getAbsolutePath());
                            CSVParser parser = new CSVParser(fileReader, CSVFormat.RFC4180
                                    .withHeader(this.headersCC())
                                    .withIgnoreEmptyLines()
                                    .withDelimiter(';')
                                    .withFirstRecordAsHeader());
                            fileLenght=parser.getRecords().size();
                            fileReader.close();
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
    String[] headersCC() {
        return new String[]{TYPE_MESSAGE, MODULE, BIC_DONNEUR_ORDER, NUM_CHRONO, MESSAGE_20, STATUT, DATE_INTEGRATION,
                DATE_VALEUR, CODE_BIC_BANQUE, RIB_BENEFICIAIRE, NOM_BENEFICIAIRE, MESSAGE_52A, NATURE_FRAIS, MONTANT_SANS_FRAIS, MOTIF_OPERATION};
    }
    public void moveFile(String filePath, String target) {
        Path sourcePath = Paths.get(filePath);
        Path targetPath = Paths.get(target);
        try {
            // Move the file
            Files.move(sourcePath, targetPath.resolve(sourcePath.getFileName()), StandardCopyOption.REPLACE_EXISTING);
            log.info("File with path {} moved successfully to {}", filePath, target);
        } catch (IOException e) {
            e.printStackTrace();
            log.error("Error moving the file {} to {}", filePath, target);
        }
    }
}
