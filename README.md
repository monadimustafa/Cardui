countsTask : any;
  statusTask : any;
  getchart(param: Application):any {
    this.countsTask  = [];
    this.statusTask = [];
    let compt = 0;
    let data = null;

    param.tasks.forEach(request => {
      let count = 0;
      let stat = "";
      count = request.count;
      stat = request.status;
      compt += count;
      this.countsTask.push(count);
      this.statusTask.push(stat);
    });

     data = this.statusTask.map((status : string, index: number) => ({
      name: status,
      y: this.countsTask[index],
    }));

    this.chartOptions = {
      title: {
        text: 'Total<br><span class="countTask">'+compt+' Tâches</span>',
        align: 'center',
        y: 60,
      },
      plotOptions: {
        pie: {
          innerSize: '95%',
          size: '130px',
          slicedOffset : 50,
          borderWidth: 0,
          dataLabels: {
            enabled: false,
            format: '<b>{point.name}</b>: {point.y}'
          }
        }
      },
      series: [{
        type: 'pie',
        name: 'Tache ',
        data: data
      }],
    };
    return this.chartOptions;
  }

  Highcharts: typeof Highcharts = Highcharts;
  chartOptions: Highcharts.Options = {};
