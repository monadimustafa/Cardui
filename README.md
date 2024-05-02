<div class="chart-container">
  <highcharts-chart
    [Highcharts]="Highcharts"
    [options]="splineChartOptions"
    style="width: 100%; height: 400px;"
  ></highcharts-chart>
</div>

<div class="chart-container">
  <highcharts-chart
    [Highcharts]="Highcharts"
    [options]="pieChartOptions"
    style="width: 100%; height: 400px;"
  ></highcharts-chart>
</div>


import { Component, OnInit } from '@angular/core';
import * as Highcharts from 'highcharts';
import HC_exporting from 'highcharts/modules/exporting';
import HC_exportData from 'highcharts/modules/export-data';
import { UserService } from 'chemin_vers_votre_service';

HC_exporting(Highcharts);
HC_exportData(Highcharts);

@Component({
  selector: 'app-chart',
  templateUrl: './chart.component.html',
  styleUrls: ['./chart.component.css']
})
export class ChartComponent implements OnInit {
  Highcharts: typeof Highcharts = Highcharts;
  splineChartOptions: Highcharts.Options = {};
  pieChartOptions: Highcharts.Options = {};
  users: User[] = [];

  constructor(private userService: UserService) { }

  ngOnInit(): void {
    this.userService.getUsers().subscribe((data: User[]) => {
      this.users = data;
      this.prepareChartData();
    });
  }

  prepareChartData(): void {
    // Préparation des données pour le graphique en courbes (spline)
    const splineCategories = this.users.map(user => user.firstName + ' ' + user.lastName);
    const splineData = this.users.map(user => user.applications.length);

    // Préparation des données pour le graphique à secteurs (pie)
    const portalData = this.users.reduce((acc, user) => {
      user.applications.forEach(app => {
        acc[app.appName] = (acc[app.appName] || 0) + 1;
      });
      return acc;
    }, {});

    const pieData = Object.keys(portalData).map(appName => ({
      name: appName,
      y: portalData[appName]
    }));

    // Définition des options pour les graphiques
    this.splineChartOptions = {
      chart: {
        type: 'spline'
      },
      title: {
        text: 'Nombre d\'applications par utilisateur'
      },
      xAxis: {
        categories: splineCategories
      },
      yAxis: {
        title: {
          text: 'Nombre d\'applications'
        }
      },
      series: [{
        name: 'Applications',
        data: splineData
      }]
    };

    this.pieChartOptions = {
      chart: {
        type: 'pie'
      },
      title: {
        text: 'Répartition des applications par utilisateur'
      },
      series: [{
        name: 'Applications',
        data: pieData
      }]
    };
  }
}
