this.userService.getUsers().subscribe((data: User[]) => {
      const categories = data.map(user => user.firstName + ' ' + user.lastName);
      const dataPoints = data.map(user => user.applications.length);

      this.chartOptions = {
        chart: {
          type: 'column'
        },
        title: {
          text: 'Nombre d\'applications par utilisateur'
        },
        xAxis: {
          categories: categories
        },
        yAxis: {
          title: {
            text: 'Nombre d\'applications'
          }
        },
        series: [{
          name: 'Applications',
          data: dataPoints
        }]
      };
    });
  }
