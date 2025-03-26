@Scheduled(cron = "${rpa-portal.seed.orchestrator.cron}")
    public void checkInputDirectoriesForNewFiles() throws IOException, InvalidFormatException {
        log.info("[FileWatcher - checkInputDirectoriesForNewFiles] start");
        for (String directory : directories) {
            File[] files = new File(directory).listFiles();
            if (files != null) {
                for (File file : files) {
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
            }
        }
        log.info("[FileWatcher - checkInputDirectoriesForNewFiles] End");
    }
