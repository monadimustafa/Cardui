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
  users: User[] = [];

  constructor(private userService: UserService) { }

  ngOnInit(): void {
    this.userService.getUsers().subscribe((data: User[]) => {
      this.users = data;
      // Vous pouvez maintenant utiliser les données pour créer vos graphiques Highcharts
      this.createSplineChart();
      this.createPieChart();
    });
  }

  createSplineChart(): void {
    Highcharts.chart('splineChart', {
      chart: {
        type: 'spline'
      },
      title: {
        text: 'Nombre d\'applications par utilisateur'
      },
      xAxis: {
        categories: this.users.map(user => user.firstName + ' ' + user.lastName)
      },
      yAxis: {
        title: {
          text: 'Nombre d\'applications'
        }
      },
      series: [{
        name: 'Applications',
        data: this.users.map(user => user.applications.length)
      }]
    });
  }

  createPieChart(): void {
    const portalData = this.users.reduce((acc, user) => {
      user.applications.forEach(app => {
        acc[app.appName] = (acc[app.appName] || 0) + 1;
      });
      return acc;
    }, {});

    const data = Object.keys(portalData).map(appName => ({
      name: appName,
      y: portalData[appName]
    }));

    Highcharts.chart('pieChart', {
      chart: {
        type: 'pie'
      },
      title: {
        text: 'Répartition des applications par utilisateur'
      },
      series: [{
        name: 'Applications',
        data: data
      }]
    });
  }
}
