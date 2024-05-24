import { Component, OnInit } from '@angular/core';
import * as Highcharts from 'highcharts';
import { DataService } from './data.service'; // Adjust path as necessary

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  Highcharts: typeof Highcharts = Highcharts; // required
  chartOptions: Highcharts.Options;

  constructor(private dataService: DataService) {}

  ngOnInit() {
    const countsTask = this.dataService.getCountsTask();
    const statusTasks = this.dataService.getStatusTasks();

    const data = statusTasks.map((status, index) => ({
      name: status,
      y: countsTask[index]
    }));

    this.chartOptions = {
      chart: {
        type: 'pie'
      },
      title: {
        text: 'Task Status Counts'
      },
      plotOptions: {
        pie: {
          innerSize: '50%',
          depth: 45
        }
      },
      series: [{
        type: 'pie',
        name: 'Tasks',
        data: data
      }]
    };
  }
}
-------------------------------------------------

import { Component, OnInit } from '@angular/core';
import * as Highcharts from 'highcharts';
import { DataService } from './data.service'; // Adjust path as necessary

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  Highcharts: typeof Highcharts = Highcharts; // required
  chartOptions: Highcharts.Options;

  constructor(private dataService: DataService) {}

  ngOnInit() {
    const countsTask = this.dataService.getCountsTask();
    const statusTasks = this.dataService.getStatusTasks();

    const data = statusTasks.map((status, index) => ({
      name: `${status}: ${countsTask[index]}`,
      y: countsTask[index]
    }));

    this.chartOptions = {
      chart: {
        type: 'pie'
      },
      title: {
        text: 'Task Status Counts'
      },
      plotOptions: {
        pie: {
          innerSize: '50%',
          depth: 45,
          dataLabels: {
            enabled: true,
            format: '<b>{point.name}</b>: {point.y}'
          }
        }
      },
      series: [{
        type: 'pie',
        name: 'Tasks',
        data: data
      }],
      legend: {
        align: 'right',
        verticalAlign: 'middle',
        layout: 'vertical',
        itemMarginTop: 10,
        itemMarginBottom: 10,
        labelFormatter: function() {
          return this.name;
        }
      }
    };
  }
}
---------------------------
import { Component, OnInit } from '@angular/core';
import * as Highcharts from 'highcharts';
import { DataService } from './data.service'; // Adjust path as necessary

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  Highcharts: typeof Highcharts = Highcharts; // required
  chartOptions: Highcharts.Options;

  constructor(private dataService: DataService) {}

  ngOnInit() {
    const countsTask = this.dataService.getCountsTask();
    const statusTasks = this.dataService.getStatusTasks();

    const data = statusTasks.map((status, index) => ({
      name: status,
      y: countsTask[index]
    }));

    this.chartOptions = {
      chart: {
        type: 'pie'
      },
      title: {
        text: 'Task Status Counts'
      },
      plotOptions: {
        pie: {
          innerSize: '50%',
          depth: 45,
          dataLabels: {
            enabled: true,
            format: '<b>{point.name}</b>: {point.y}'
          }
        }
      },
      series: [{
        type: 'pie',
        name: 'Tasks',
        data: data
      }],
      legend: {
        align: 'right',
        verticalAlign: 'middle',
        layout: 'vertical',
        itemMarginTop: 10,
        itemMarginBottom: 10,
        labelFormatter: function() {
          const status = this.name;
          const count = this.y;
          return `<a href="https://example.com/status/${status}" target="_blank">${status}: ${count}</a>`;
        },
        useHTML: true
      }
    };
  }
}
