<template>
	<div>
	<!-- breadcrumb -->
				<div class="breadcrumb-header justify-content-between">
					<div class="left-content">
						<div>
						  <h2 class="main-content-title tx-24 mg-b-1 mg-b-lg-1">ລາຍງານ</h2>
					
						</div>
					</div>
					<div class="main-dashboard-header-right">
						
					</div>
				</div>
				<!-- breadcrumb -->
    <div class="row row-sm">
        <div class="col-md-8">
            <div class="card mg-b-20">
							<div class="card-header pb-0">
								<div class="d-flex justify-content-between">
									<h4 class="card-title mg-b-0">ລະບົບລາຍ</h4>
									<i class="mdi mdi-dots-horizontal text-gray"></i>
								</div>
								<div class="row pd-b-10 justify-content-end">
                                            <div class="btn-group me-2" role="group" aria-label="Basic example" style="width:180px;"> 
                                                <button type="button" class="btn btn-secondary" @click="monthtype = 'm'"> <i class="mdi mdi-menu-right" v-if="monthtype == 'm'"></i> ເດືອນ</button>
                                                <button type="button" class="btn btn-secondary" @click="monthtype = 'y'"> <i class="mdi mdi-menu-right" v-if="monthtype == 'y'"></i> ປີ</button>
                                            </div>
                                            <input type="date" style="width: 180px;" v-model="dmy" class="form-control me-2">
                                            <button class="btn btn-success text-white me-2" @click="CreateReport()"  style="width:180px;">
                                                <i class="mdi mdi-view-list"></i> ສະແດງລາຍງານ
                                            </button>
                                        </div>
							</div>
							<div class="card-body">
								<linechart
								:chartData="chdata"
								:chartOptions="chdataop"
								:update="update_chart"
								:key="key"
								v-if="shchart"
								></linechart>
							</div>
						</div>
        </div>
        <div class="col-md-4">
            <div class="row">
            <div class="col-12">
						<div class="card bg-primary-gradient text-white ">
							<div class="card-body">
								<div class="row">
									<div class="col-6">
										<div class="icon1 mt-2 text-center">
											<i class="ti-stats-up tx-40"></i>
										</div>
									</div>
									<div class="col-6">
										<div class="mt-0 text-center">
											<span class="text-white">ລາຍຮັບ</span>
											<h2 class="text-white mb-0">{{ formatPrice(sum_income) }} ກີບ</h2>
										</div>
									</div>
								</div>
							</div>
						</div>
					</div>
                    <div class="col-12">
						<div class="card bg-danger-gradient text-white">
							<div class="card-body">
								<div class="row">
									<div class="col-6">
										<div class="icon1 mt-2 text-center">
											<i class="ti-pie-chart tx-40"></i>
										</div>
									</div>
									<div class="col-6">
										<div class="mt-0 text-center">
											<span class="text-white">ລາຍຈ່າຍ</span>
											<h2 class="text-white mb-0">{{ formatPrice(sum_expense) }} ກີບ</h2>
										</div>
									</div>
								</div>
							</div>
						</div>
					</div>
                    <div class="col-12">
						<div class="card bg-success-gradient text-white">
							<div class="card-body">
								<div class="row">
									<div class="col-6">
										<div class="icon1 mt-2 text-center">
											<i class="fe fe-bar-chart-2 tx-40"></i>
										</div>
									</div>
									<div class="col-6">
										<div class="mt-0 text-center">
											<span class="text-white">ກຳໄລ</span>
											<h2 class="text-white mb-0">{{ formatPrice(sum_profit) }} ກີບ</h2>
										</div>
									</div>
								</div>
							</div>
						</div>
					</div>
                </div>
        </div>
    </div>
	</div>
</template>

<script>

	import linechart from "../components/LineChart.vue";
	import moment from "moment";

export default {
    name: 'Minipos02Report',

    data() {
        return {
			shchart:false,
            monthtype:'m',
			dmy:'',
			data_income:[],
			data_expense:[],
			key:0,
			update_chart:null,
            chdata:[],
            chdataop:{
                 responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                    yAxes: [
                        {
                        ticks: {
                            display: true,
                            beginAtZero: false,
                            callback: function (value, index, values) {
                            return ( Number(value) .toFixed(0) .replace(/./g, function (c, i, a) { return i > 0 && c !== "." && (a.length - i) % 3 === 0 ? "," + c : c; }) + " ກີບ" );
                            },
                        },
                        gridLines: {
                            show: true,
                        },
                        },
                    ],
                    },
                    tooltips: {
                    callbacks: {
                        label: function (tooltipItem, data) {
                        return (
                            Number(tooltipItem.yLabel) .toFixed(0) .replace(/./g, function (c, i, a) { return i > 0 && c !== "." && (a.length - i) % 3 === 0 ? "," + c : c; }) + " ກີບ" );
                        },
                    },
                    mode: "index",
                    intersect: false,
                    },
            },
        };
    },
	components:{
		linechart,moment
	},
	computed:{
		sum_income(){
			return this.data_income.reduce((num, item) => num + item.price, 0 );
		},
		sum_expense(){
			return this.data_expense.reduce((num, item) => num + item.price, 0 );
		},
		sum_profit(){
			return this.sum_income-this.sum_expense
		}
	},
    mounted() {
        
    },

    methods: {
		formatPrice(value) {
			let val = (value / 1).toFixed(0).replace(",", ".");
			return val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
		},
        CreateReport(){

			this.$axios.get("/sanctum/csrf-cookie").then((response) => {
				this.$axios.post("/api/report", {
				monthtype: this.monthtype,
				dmy: this.dmy,
				})
				.then((response) => {
					//console.log(response.data)
					this.data_income = response.data.income;
					this.data_expense = response.data.expense;
					this.GenGrap();
				})
				.catch((error) => {
					this.loading = false;
                });
            });
		},
		GenGrap(){
			
			   this.shchart = true;

				if (this.monthtype == "m") { //ກວດຊອບ ການສະແດງຜົນ ເປັນເດືອນ
                    
					this.key++;
                    let re_income = [];
                    let re_expense = [];

                    let y = this.dmy.split("-")[0];
                    let m = this.dmy.split("-")[1];
                    let lastday = this.Getlastday(y, m);

                    //console.log('ຊອກວັນທີ່: '+lastday)

                    let chart_label = [];
                    for (let i = 0; i < lastday; i++) {
                    chart_label.push("ວັນທີ່ " + (i + 1));
                    //console.log(i)
                    }

                    re_income = this.Get_data_chart(lastday, this.data_income) || 0;
                    re_expense = this.Get_data_chart(lastday, this.data_expense) || 0;
                

                    this.chdata = {
                    labels: chart_label,
                    datasets: [
                        {
                        label: "ລາຍຮັບ",
                        fill: false,
                        borderColor: "#3366CC",
                        data: re_income,
                        },
                        {
                        label: "ລາຍຈ່າຍ",
                        fill: false,
                        borderColor: "#DC3912",
                        data: re_expense,
                        },
						

                    ],
                    };
                    this.update_chart = Math.floor(Math.random() * 100);
                    // ສ້າງກຼາບໃນຮູບແບບຂອງປີ
                } else if (this.monthtype == "y") { //ກວດຊອບ ການສະແດງຜົນ ເປັນປີ
                    this.key++;
                    let re_income = [];
                    let re_expense = [];
                    let chart_label = ["ເດືອນ 1","ເດືອນ 2","ເດືອນ 3","ເດືອນ 4","ເດືອນ 5","ເດືອນ 6","ເດືອນ 7","ເດືອນ 8","ເດືອນ 9","ເດືອນ 10","ເດືອນ 11","ເດືອນ 12",];

                    re_income = this.Get_data_chart_y(12, this.data_income) || 0;
                    re_expense = this.Get_data_chart_y(12, this.data_expense) || 0;

                    this.chdata = {
                    labels: chart_label,
                    datasets: [
                        {
                        label: "ລາຍຮັບ",
                        fill: false,
                        borderColor: "#3366CC",
                        data: re_income,
                        },
                        {
                        label: "ລາຍຈ່າຍ",
                        fill: false,
                        borderColor: "#DC3912",
                        data: re_expense,
                        },
                    ],
                    };
                    this.update_chart = Math.floor(Math.random() * 100);
                }
		},
		Getlastday(y, m) {
			let dd = new Date(y, m , 0).getDate();
			//console.log('ວັນທີ່ສຸດທ້າຍ ຂອງເດືອນ:'+dd)
			return dd
		},
		Getday(value) {
		return moment(value).format("DD");
		},
		Getmonth(value) {
		return moment(value).format("MM");
		},
		Getyear(value) {
		return moment(value).format("YYYY");
		},
		Get_data_chart(date, data) {
		///  console.log(data)
		let new_db_in = [];
		let databack = [];
		for (let y = 0; y < data.length; y++) {
			if (data[y] != "") {
			let day = this.Getday(data[y].created_at);
			new_db_in.push({ price: data[y].price, day: day });
			}
		}
		// console.log(new_db_in)
		
			let update_db_in = new_db_in.reduce((a, c) => {
				let filtered = a.filter((el) => el.day === c.day);
				if (filtered.length > 0) {
				a[a.indexOf(filtered[0])].price =
					parseInt(a[a.indexOf(filtered[0])].price) + parseInt(c.price);
				} else {
				a.push(c);
				}
				return a;
			}, []);
		 // console.log(update_db_in)
			for (let i = 0; i < date; i++) {
				let type = true;
				for (let k = 0; k < update_db_in.length; k++) {
				if (update_db_in[k].day == i + 1) {
					if (type) {
					databack.push(update_db_in[k].price);
					type = false;
					}
				}
				}
				if (type) {
				databack.push(0);
				type = false;
				}
			}
		return databack;
		},
		Get_data_chart_y(monthchart, data) {
		let new_db_in = [];
		let databack = [];
		for (let y = 0; y < data.length; y++) {
			if (data[y] != "") {
			let month = this.Getmonth(data[y].created_at);
			new_db_in.push({ price: data[y].price, month: month });
			}
		}

		let update_db_in = new_db_in.reduce((a, c) => {
			let filtered = a.filter((el) => el.month == c.month);
			if (filtered.length > 0) {
			a[a.indexOf(filtered[0])].price =
				parseInt(a[a.indexOf(filtered[0])].price) + parseInt(c.price);
			} else {
			a.push(c);
			}
			return a;
		}, []);

		for (let i = 0; i < monthchart; i++) {
			let type = true;
			for (let k = 0; k < update_db_in.length; k++) {
			if (update_db_in[k].month == i + 1) {
				if (type) {
				databack.push(update_db_in[k].price);
				type = false;
				}
			}
			}
			if (type) {
			databack.push(0);
			type = false;
			}
		}

		return databack;
		},
    },
	beforeRouteEnter(to, from, next) {
		//window.Laravel.urlpath = to.name
    if (!window.Laravel.isLoggedin) {
      window.location.href = "/login";
    }
    next();
  },
};
</script>

<style lang="scss" scoped>

</style>