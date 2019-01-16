<template>
    <Page>
        <ActionBar title="Gate Keeper App v.0.1" android:flat="true"/>
        <TabView android:tabBackgroundColor="#53ba82"
                 android:tabTextColor="#c4ffdf"
                 android:selectedTabTextColor="#ffffff"
                 androidSelectedTabHighlightColor="#ffffff">
            <TabViewItem title="QR Scanner">
                <GridLayout columns="*" rows="*">
                    <Button text="Open Scanner" @tap="onButtonTap()"/>
                </GridLayout>
            </TabViewItem>
            <TabViewItem title="Student Check In List">
                <GridLayout columns="*" rows="*">
                    <ListView for="stud in checkins">
                        <v-template>
                            <Label :text="'ID: ' + stud.id + ', @' + stud.checkedInAt"/>
                        </v-template>
                    </ListView>
                </GridLayout>
            </TabViewItem>
            <TabViewItem title="Tab 3">
                <GridLayout columns="*" rows="*">
                    <Label class="message" text="Tab 3 Content" col="0" row="0"/>
                </GridLayout>
            </TabViewItem>
        </TabView>
    </Page>
</template>

<script>
    import axios from "axios"
    var BarcodeScanner = require("nativescript-barcodescanner").BarcodeScanner
    var barcodescanner = new BarcodeScanner()

  export default {
    methods: {
        onButtonTap: function() {
            var self = this
            barcodescanner.scan({
                formats: "QR_CODE",
                cancelLabel: "EXIT. Also, try the volume buttons!",
                cancelLabelBackgroundColor: "#333333",
                message: "Use the volume buttons for extra light",
                showFlipCameraButton: true,
                preferFrontCamera: false,
                beepOnScan: true,
                openSettingIfPermissionWasPreviouslyDenied: true,
                closeCallback: function() {
                    console.log('Scanner closed')
                }
            }).then(
                function(result) {
                    barcodescanner.stop()
                    console.log("Scan format: " + result.format)
                    console.log("Scan text: " + result.text)
                    axios({method: "GET", "url": "http://192.168.1.101:5000/api/" + result.text}).then((resp)=>{
                        self.scannedAt = resp.data.data
                        self.checkins.push({
                            id: result.text,
                            checkedInAt: self.scannedAt
                        })
                        alert({
                            title: 'Check-in Succeeded',
                            message: result.text + " checked in at " + self.scannedAt,
                            okButtonText: 'Close'
                        })
                    })
                },
                function(error) {
                    alert({
                        title: 'Check-in Failed',
                        message: 'Please try again or contact the system admin.'
                    })
                }
            )
        }
    },
    data() {
      return {
        msg: 'Hello Vue World!',
        scannedAt: undefined,
        checkins: []
      }
    },
  }
</script>

<style scoped>
    ActionBar {
        background-color: #53ba82;
        color: #ffffff;
    }

    .message {
        vertical-align: center;
        text-align: center;
        font-size: 20;
        color: #333333;
    }
</style>
