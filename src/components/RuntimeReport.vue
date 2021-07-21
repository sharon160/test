<template>
  <div>
      <v-dialog v-model="dialog" max-width="500px">
        <template v-slot:activator="{ on }">
            <v-btn class="white--text" color="#304ffe" v-on="on">
                <v-icon size="14" class="mx-1">fas fa-search</v-icon>
            {{ 'Filter' }}</v-btn>
        </template>
        <v-card>
            <v-card-title style="background-color: #DCDCDC">
                <span class="headline">{{ 'Filter Report' }}</span>
            </v-card-title>
            <v-form ref="form">
            <v-card-text>
                <v-container text-xs-center grid-list-md>
                    <div>
                        <v-select
                            :rules="fieldRules"
                            :items="['ON', 'OFF']"
                            v-model="filter.field"
                            :label="'Field'"
                        ></v-select>
                    </div>

                    <div>
                        <v-select
                            :rules="expressionRules"
                            :items="['<','>']"
                            v-model="filter.expression"
                            :label="'Expression'"
                        ></v-select>
                    </div>

                    <div>
                        <v-text-field
                            v-model="filter.hour"
                            :label="'Hour'"
                        ></v-text-field>
                    </div>
                </v-container>
            </v-card-text>
            <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="red" text @click="clear">{{ 'Clear'}}</v-btn>
                <v-btn color="blue darken-1" text @click="apply">{{ 'Apply' }}</v-btn>
            </v-card-actions>
        </v-form>
        </v-card>
    </v-dialog>
    <div class="form-data">
        <div class="custom-row">
            <h4 class="report-form-item">{{'Group'}}:</h4>
            <v-select
                :items="groupTitlesList"
                menu-props="auto"
                :label="'All'"
                hide-details
                single-line
                v-model="groupSelected"
                clearable
            ></v-select>
        </div>
        <div class="custom-row" style="padding-top: 20px">
            <h4 class="report-form-item">{{'Pick Date'}}:</h4>
            <div class="custom-row">
                <input class="date-input" v-model="date" type="date" />
            </div>
        </div>
        <div style="padding-top: 10px">
            <v-btn
                @click="generateReport"
                color="success"
            >{{'Generate'}}</v-btn>
        </div>
    </div>
    <div id="viewerContent"></div>
  </div>
</template>

<script>
import { Stimulsoft } from 'stimulsoft-reports-js/Scripts/stimulsoft.designer'

export default {
  name: "RuntimeReport",
  data() {
    return {
        groupTitlesList: ['All'],
        groupSelected: '',
        date: '',
        reportData: [],
        filteredReportData: [],
        dialog: false,
        filter: {
            field: '',
            expression: '',
            hour: ''
        },
        // hourRules: [
        //     (h) => !!h || 'Hour is required',
        //     (h) => (h && moment(h, 'HH:mm', true).isValid()) || 'Please enter valid hour (HH:MM)'
        // ],
        fieldRules: [
            (f) => !!f || 'Field is required'
        ],
        expressionRules: [
            (e) => !!e || 'Expression is required'
        ],
        monthNames: ['January', 'February', 'March', 'April', 'May', 'June',
        'July', 'August', 'September', 'October', 'November', 'December'
        ]
    }
  },

  async mounted() {
  },
  methods: {
      async generateReport() {
        this.reportData = [];

        const date = new Date(this.date);
        const periodFrom = new Date(date.getFullYear(), date.getMonth(), date.getDate(), 12);
        const periodTo = new Date(periodFrom.getFullYear(), periodFrom.getMonth(), periodFrom.getDate() + 1, periodFrom.getHours(), periodFrom.getMinutes(), periodFrom.getSeconds() - 1);

        const streamName = 'clean_power';

        const reports = [
            {
                name: 'T_Ashdod_000003_44:Dd:92:C5:68:B8',
                id: '7ac615b292d54faea8058ad014589c3b'
            },
            {
                name: 'T_Ashdod_000004_Dc:88:0f:10:6e:37',
                id: '5d3210d7f0ca4edd82e0651696339ed2'
            },
            {
                name: 'T_Ashdod_000005_Ee:54:99:88:Fc:E8',
                id: 'cac306b0bbed4f208ce30d060d8e668d'
            }
        ]

        await Promise.all(reports.map(async (device) => {
            const response = await this.getStreamHistory(device.id, streamName, { from: periodFrom.getTime(), to: periodTo.getTime() });
            this.responseIteration(response, device);
        }));

        this.filteredReportData = this.reportData;

        var report = Stimulsoft.Report.StiReport.createNewReport();
        report.loadFile("runtimeReport.mrt");
        // var cultureName = "he-IL";
        // report.localizeReport(cultureName);

        var dataSet = new Stimulsoft.System.Data.DataSet("results");
        dataSet.readJson(JSON.stringify(this.filteredReportData));
        report.dictionary.databases.clear();
        report.regData("results", "results", dataSet);
        report.dictionary.synchronize();

        report.renderAsync();

        var options = new Stimulsoft.Viewer.StiViewerOptions();
        options.appearance.rightToLeft = true;

        // console.log(report.getComponents());
        // var databand = report.renderedPages.getComponentByName("Tableroot");
        // databand.Filters.Add(new Stimulsoft.StiFilter(`root.${filterBy} ${expression} ${value}`));
        var viewer = new Stimulsoft.Viewer.StiViewer(options, "StiViewer", false);
        viewer.report = report;
        viewer.renderHtml("viewerContent");
      
    },

    async getStreamHistory(deviceId, streamName, period) {
        const headers = {
            'x-api': '84f7c0ca13e24e01b0a4026b725f8af1', // ashdod
            'Content-Type': 'application/json'
        }
        const dataRequest = new Request(
            `https://api.cloudofthings.com/v3/devices/${deviceId}/streams/${deviceId}.${streamName}/history/?from=${period.from}&to=${period.to}`,
            {
                headers: headers
            }
        );

        const response = await fetch(dataRequest);
        const data = await response.json();
        return data;
    },

    responseIteration(response, device) {
        let sum = 0;

        if (!Array.isArray(response)) {
            return;
        }
        if (response.length === 0)
            return;

        const firstTime = response[0][0];
        const firstValue = response[0][1];

        const lastTime = new Date(response[response.length - 1][0]);
        if (lastTime.getHours() >= 12) {
            lastTime.setDate(lastTime.getDate() + 1);
        }
        lastTime.setHours(11, 59, 59);
        const lastValue = response[response.length - 1][1];
        response.push([lastTime, lastValue]);


        let prevTime = response[0][0];
        let prevvalue = response[0][1];


        let onEntries = '';
        let dimEntries = '';
        let offEntries = '';
        if (firstValue === 0) {
            offEntries += `${new Date(firstTime).toLocaleTimeString('he-IL', { hour: '2-digit', minute: '2-digit'})}`;
        }
        else if (firstValue > 0) {
            onEntries += `${Math.round(firstValue / 254 * 100)}% ${new Date(firstTime).toLocaleTimeString('he-IL', { hour: '2-digit', minute: '2-digit'})}`;
        }
        response.slice(1).forEach((element) => {
            const tempDate = new Date(element[0]);
            if (prevvalue === 0 && element[1] > 0 && !onEntries) {
                onEntries += ` ${Math.round(element[1] / 254 * 100)}% ${tempDate.toLocaleTimeString('he-IL', { hour: '2-digit', minute: '2-digit'})}`;
            }
            else if (prevvalue === 0 && element[1] > 0 && onEntries) {
                dimEntries += `${Math.round(element[1] / 254 * 100)}% ${tempDate.toLocaleTimeString('he-IL', { hour: '2-digit', minute: '2-digit'})} | `;
            }
            if (prevvalue > 0 && element[1] > 0 && prevvalue !== element[1]) {
                dimEntries += `${Math.round(element[1] / 254 * 100)}% ${tempDate.toLocaleTimeString('he-IL', { hour: '2-digit', minute: '2-digit'})} | `;
            }
            if (prevvalue > 0 && element[1] === 0) {
                dimEntries += `${Math.round(element[1] / 254 * 100)}% ${tempDate.toLocaleTimeString('he-IL', { hour: '2-digit', minute: '2-digit'})} | `;
            }
            if (element[1] > 0 && prevvalue > 0 || element[1] === 0 && prevvalue > 0) {
                sum += (element[0] - prevTime) / 1000;
            }
            prevTime = element[0];
            prevvalue = element[1];
        });
        if (response[response.length - 1][1] === 0) {
            const dimEntriesReverse = dimEntries.split('').reverse().join('');
            const firstIndex = dimEntriesReverse.indexOf('|');
            let lastOffEntry = '';
            let lastIndex = dimEntriesReverse.slice(firstIndex + 1).indexOf('|');

            lastOffEntry = dimEntriesReverse.slice(firstIndex + 1, firstIndex + lastIndex + 1).split('').reverse().join('');

            dimEntries = dimEntries.replace(lastOffEntry + '|', '');
            offEntries = lastOffEntry.replace(/\s/g, '').slice(lastOffEntry.indexOf('%'));

            if (lastIndex === -1) {
                lastIndex = dimEntriesReverse.length - 1;
                lastOffEntry = dimEntriesReverse.slice(firstIndex + 1, firstIndex + lastIndex + 1).split('').reverse().join('');

                dimEntries = dimEntries.replace(lastOffEntry, '');
                offEntries = lastOffEntry.replace(/\s/g, '').slice(lastOffEntry.indexOf('%') + 1);
            }
        }
        let onHours = '';
        onHours = new Date(sum * 1000).toISOString().substr(11, 5);

        const info = {
            name: device.name,
            // poleNumber: JSON.parse(device['meta.device']) ? JSON.parse(device['meta.device'])['Pole Number'] || '-' : '-',
            // cabinetNumber: JSON.parse(device['meta.device']) ? JSON.parse(device['meta.device'])['Cabinet ID'] || '-' : '-',
            on: onEntries,
            dim: dimEntries,
            off: offEntries,
            dailyRuntime: onHours
        };
        this.reportData.push(info);
    },
    async clear() {
        this.$refs.form.reset();
        this.filteredReportData = this.reportData;
        const reportResponse = await fetch("RuntimeReport.rdlx-json");
        const report = await reportResponse.json();
        report.DataSources[0].ConnectionProperties.ConnectString = "jsondata=" + JSON.stringify(this.filteredReportData);
        const viewer = this.$refs.reportViewer.Viewer();
        viewer.open(report, { ReportParams: [{ Name: "group", Value: [this.groupSelected] }, { Name: "date", Value: [this.date] }, { Name: "project", Value: ['ff9875dd75224a8aa292bb2b33b1e110']}, { Name: "apiKey", Value: ['84f7c0ca13e24e01b0a4026b725f8af1'] }]});
    },
    async apply() {
        const valid = this.$refs.form.validate();
        if (valid) {
            if (this.filter.expression === '>') {
                this.filteredReportData = this.reportData
                    .filter((report) => report[this.filter.field.toLowerCase()]
                    .slice(report[this.filter.field.toLowerCase()].indexOf('%') + 1).replace(/\s/g, '') > this.filter.hour);
            }
            else if (this.filter.expression === '<') {
                this.filteredReportData = this.reportData.filter((report) => report[this.filter.field.toLowerCase()].replace(/\s/g, '') < this.filter.hour);
            }
            this.dialog = false;
            const reportResponse = await fetch("RuntimeReport.rdlx-json");
            const report = await reportResponse.json();
            report.DataSources[0].ConnectionProperties.ConnectString = "jsondata=" + JSON.stringify(this.filteredReportData);
            const viewer = this.$refs.reportViewer.Viewer();
            viewer.open(report, { ReportParams: [{ Name: "group", Value: [this.groupSelected] }, { Name: "date", Value: [this.date] }, { Name: "project", Value: ['ff9875dd75224a8aa292bb2b33b1e110']}, { Name: "apiKey", Value: ['84f7c0ca13e24e01b0a4026b725f8af1'] }]});
        }
    }
  }
};
</script>

<style src="../../node_modules/stimulsoft-reports-js/Css/stimulsoft.viewer.office2010.blue.css"></style>


<style>
#viewer-host {
  width: 100%;
  height: 100vh;
}

#designer-host {
  width: 100%;
  height: 100vh;
}

.gc-btn--accent {
    color: white !important;
}
</style>