import {Component, OnInit} from '@angular/core';
import * as Highcharts from 'highcharts';
import {DashboardService} from "../../service/dashboard.service";
import {User} from "../../../../core/models/user.model";
@Component({
  selector: 'portal-agregateur-pie-chart',
  templateUrl: './pie-chart.component.html',
  styleUrls: ['./pie-chart.component.scss']
})
export class PieChartComponent implements OnInit{
  Highcharts: typeof Highcharts = Highcharts;
  chartOptions: Highcharts.Options = {};
  users!: [];
  totalApps!: [];
  constructor(private serviceUser : DashboardService) {
  }
  ngOnInit(): void {
    this.getData();
  }
  getData(){
    this.serviceUser.getUsers().subscribe((data: User[]) => {
      this.users = data.map(user => user.firstName + ' ' + user.lastName) as [];
      this.totalApps = data.map(user => user.applications?.length) as [];
      console.log(this.users)
      console.log(this.totalApps)
      this.chartOptions = {
        chart: {
          type: 'column'
        },
        title: {
          text: 'Nombre d\'applications par utilisateur'
        },
        xAxis: {
          title:{
            text : 'Utilisateurs'
          },
          categories: this.users
        },
        yAxis: {
          title: {
            text: 'Nombre d\'applications',
          },
        },
        series: [{
          data: this.users,
          type : 'column'
        }]
      };
    });
  }
  /*this.getData();
      this.chartOptions = {
        chart: {
          type: 'column',
        },
        title: {
          text: 'Nombre d\'applications par utilisateur',
        },
        xAxis: {
          categories: this.users
        },
        yAxis: {
          title: {
            text: 'Nombre d\'applications',
          },
        },
        series: [{
          data: this.totalApps,
          type: 'line'
        }]
      };

  }

  getData(){
    this.serviceUser.getUsers().subscribe((data: User[]) => {
      this.users = data.map(user => user.firstName + ' ' + user.lastName);
      this.totalApps = data.map(user => user.applications?.length) as number[];
      console.log(this.users)
      console.log(this.totalApps)
    });
  }*/
}
