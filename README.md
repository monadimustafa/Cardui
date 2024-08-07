import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { HttpClient } from '@angular/common/http';
import { catchError, map, of, startWith } from 'rxjs';

@Component({
  selector: 'app-file-upload',
  templateUrl: './file-upload.component.html',
  styleUrls: ['./file-upload.component.scss']
})
export class FileUploadComponent implements OnInit {
  loadFileFormGroup: FormGroup;
  isUploading = false;
  fileName = '';
  fileSize = '';
  progress = 0;
  selectedFile: File | null = null;
  sendFile = false;
  sccRequest$: any;

  constructor(private fb: FormBuilder, private http: HttpClient) {}

  ngOnInit(): void {
    this.initProcessFormGroup();
  }

  onFileSelected($event: any) {
    this.selectedFile = $event.target.files[0];
    if (this.selectedFile) {
      this.fileSize = (this.selectedFile.size / 1024 / 1024).toFixed(2) + ' MB';
      this.fileName = this.selectedFile.name;
    }
  }

  createRequest(): void {
    if (this.loadFileFormGroup.valid && this.selectedFile) {
      const requestData: DemandeClosAccountDTO = {
        clientDto: { /* client data */ },
        ribs: [/* array of RIBs */],
        motif: this.form.motif.value,
        date: new Date()
      };
      this.sendFiles(requestData);
    } else {
      console.error("Form is invalid or file is not selected");
    }
  }

  sendFiles(requestData: DemandeClosAccountDTO) {
    const formData = new FormData();
    formData.append('file', this.selectedFile!, this.selectedFile!.name);
    formData.append('demande', new Blob([JSON.stringify(requestData)], { type: 'application/json' }));

    this.http.post('/api/request-endpoint', formData).pipe(
      map(response => {
        this.initProcessFormGroup();
        this.isUploading = false;
        // Handle success
      }),
      catchError(err => {
        // Handle error
        return of(err);
      })
    ).subscribe();
  }

  get form() {
    return this.loadFileFormGroup.controls;
  }

  initProcessFormGroup() {
    this.loadFileFormGroup = this.fb.group({
      motif: ["", Validators.required]
    });
  }
}

export interface DemandeClosAccountDTO {
  clientDto?: Client;
  ribs?: string[];
  motif?: string;
  date?: Date;
}

export interface Client {
  codeClient?: string;
  nomOrRs?: string;
  cinOrRc?: string;
  agence?: string;
  rib: string;
}

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.bind.annotation.RequestPart;

@RestController
@RequestMapping("/api")
public class RequestController {

    @PostMapping("/request-endpoint")
    public ResponseEntity<?> handleFileUpload(
            @RequestPart("file") MultipartFile file,
            @RequestPart("demande") DemandeClosAccountDTO demandeClosAccountDTO) {

        // Process the file and demandeClosAccountDTO
        System.out.println("File received: " + file.getOriginalFilename());
        System.out.println("Request data: " + demandeClosAccountDTO);

        return ResponseEntity.ok("File and data received successfully");
    }
}

class DemandeClosAccountDTO {
    private Client clientDto;
    private String[] ribs;
    private String motif;
    private Date date;

    // Getters and setters
}

class Client {
    private String codeClient;
    private String nomOrRs;
    private String cinOrRc;
    private String agence;
    private String rib;

    // Getters and setters
}
