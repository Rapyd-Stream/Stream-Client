<template>
    <div class="container animate__animated animate__bounceInDown">
        <div class="row" style="padding-top: 50px;">
            <div class="col-md-12 align-t-left info-con mobile">
                <h4 class="first">What is Stream</h4>
                <div class="stream-info">
                    <p>Stream is a transfer method to <strong>stream money over time</strong> to a receiver. The funds are in a linear time based escrow (built on top of Rapyd Escrow). This is a perfect solution for contractors as they do not have to wait for their paycheck.</p>
                    <p>Streams can be cancelled at any time from the sender.</p>
                </div>
            </div>
            <div class="col-md-4 align-t-left info-con">
                <h4 class="first">What is Stream</h4>
                <div class="stream-info">
                    <p>Stream is a transfer method to <strong>stream money over time</strong> to a receiver. The funds are in a linear time based escrow (built on top of Rapyd Escrow). This is a perfect solution for contractors as they do not have to wait for their paycheck.</p>
                    <p>Streams can be cancelled at any time from the sender.</p>
                </div>
            </div>
            <div class="col-md-4 align-t-left explainer">
                <h4>Stream Creation UX</h4>
                <div class="form-widget load animate__animated animate__flipInY" v-if="this.paymentStatus === 'finish'">
                    <p><i class="fa fa-check"></i> <strong>Payment stream has been setup</strong></p>
                    <p style="margin-top: 30px;"><strong>Escrow Identifier:</strong> {{escrowId}}</p>
                    <p><strong>Payment Identifier: </strong>{{paymentId}}</p>
                </div>
                <div class="form-widget load animate__animated animate__flipInY" v-if="this.paymentStatus === 'loading'">
                    <div class="center-load">
                        <div class="lds-ring"><div></div><div></div><div></div><div></div></div>
                    </div>
                    <p class="load-info">Creating Payment Stream</p>
                </div>
                <div class="form-widget" v-if="this.paymentStatus ==='start'">
                    <form>
                        <div class="form-group">
                            <label class="firstlabel" for="amountstream">Amount To Stream (USD)</label>
                            <input v-model="amount" type="number" class="form-control" id="amountstream" placeholder="2000">
                            <small class="warn" v-if="amount < 1">Amount must be greater than 1</small>
                            <small class="warn" v-if="amount > 999">Amount must be smaller than 1000</small>
                            <p style="margin: 0; color: grey;"><small>maximum amount is 1000</small></p>
                        </div>
                        <div class="form-group">
                            <label for="durationstream" style="margin-top: 8px;">Number of days</label>
                            <input v-model="numberOfDays" min="1" type="text" class="form-control" id="durationstream" placeholder="14">
                            <small class="warn" v-if="numberOfDays < 1">Number of days must be greater than 0</small>
                        </div>
                        <div class="form-group">
                            <label for="recipientstream">Recipient</label>
                            <input type="text" disabled value="Tim Apple's Wallet" class="form-control" 
                            style="border: none; color: #555; background-color: #e8e8e8;" id="recipientstream" placeholder="">
                        </div>
                        <div class="form-group">
                            <label for="recipientstream">Payment Method</label>
                            <div class="cc-holder">
                                <!-- <i class="fa-brands fa-cc-visa enlarge-icn"></i> -->
                                <img src="../assets/visa.png" alt="" width="35px;">
                                <p>**** 1111 exp: 10/23</p>
                            </div>
                        </div>

                        <a v-if="amount < 1 || numberOfDays < 1" class="btn btn-primary dis">Submit</a>
                        <a @click="createPayment()" v-if="amount > 0 && numberOfDays > 0" class="btn btn-primary">Submit</a>
                        </form>
                </div>
            </div>
            <div class="col-md-4 col-md-4 align-t-left" v-if="streamActive">
                <h4>Stream Receiver UX</h4>
                 <div class="stream-widget">
                    <div class="info">
                        <p>A <strong>{{this.numberOfDays}} day</strong> stream of <strong>{{this.amount}} USD</strong> was opened to your wallet)</p>
                    </div>
                    <div class="currency">
                        <p class="den">USD</p>
                        <p><span>00</span>{{prettyCounter}}<span>{{prettyCounterMili}}</span></p>
                    </div>

                    <div class="prog">
                        <div class="progress" style="height: 7px;">
                        <div class="progress-bar" role="progressbar" style="width: 1%;" aria-valuenow="1" aria-valuemin="0" aria-valuemax="100"></div>
                        </div>

                        <div class="start">day 1</div>
                        <div class="end">day {{this.numberOfDays}}</div>
                    </div>

                    <div class="total">
                        <p>{{this.amount}} USD total</p>
                    </div>

                    <div class="payment-info">
                        <p>{{paymentId}}</p>
                    </div>
                </div>
            </div>

            <div class="col-md-4 align-t-left" v-if="!streamActive">
                <h4>Stream Receiver UX</h4>
                 <div class="stream-widget await">
                    <div class="info">
                        <p>Tim Apple's Wallet</p>
                    </div>

                    <div class="currency">
                        <p class="den">USD</p>
                        <p><span></span>0.00<span></span></p>
                    </div>

                    <div class="total">
                        <p>Listening for incoming streams</p>
                        <div class="lds-ripple"><div></div><div></div></div>
                    </div>

                    <div class="payment-info">
                        <p>ewallet_749c582cc44c17d1fa75ee58d3f02fd0</p>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'App',
  data: function() {
    return {
        counter: 0,
        streamActive: false,
        amount: 800,
        numberOfDays: 30,
        paymentStatus: "start",
        paymentId: "",
        escrowId: ""
    }
  },
  computed: {
    prettyCounter() {
        return this.counter.toString().slice(0,4)
    },
    prettyCounterMili() {
         return this.counter.toString().slice(4,7)
    }
  },
  components: {
    
  },
  methods: {
    async createPayment() {
        this.paymentStatus = "loading" 
        try {
            let resp = await axios.post(
            "https://rapyd-stream-backend.herokuapp.com/stream/create/",
            {
                amount: this.amount, 
                numberDays: this.numberOfDays
            },
            {
                headers: {
                "Content-type": "application/json; charset=UTF-8",
                }
            }
            )
            this.paymentStatus = "finish"
            if(resp.data.type === "success") {
                this.paymentId = resp.data.paymentId
                this.escrowId = resp.data.escrowId
            }
            this.startCount(0, this.amount, this.numberOfDays)
            this.streamActive = true
        } catch (e) {
            console.log(e, "error")
        }
    },
    startCount(current_val, final_val, days) {

    // amount still to stream
    let diff = final_val - current_val;    
    let timeLeft = (86400000 * days);

    // convert milisecons to seconds

    let resolution = 10;
    let timeLeftPerResolution = timeLeft / resolution;

    let val_per_resolution_ms_transferred = diff / timeLeftPerResolution;

    this.counter = current_val;
    const countUp = () => {
        this.counter += val_per_resolution_ms_transferred
        setTimeout(countUp, 9.9)
    }
    countUp()



      // get current value 
      // get final value
    }
  },    
  mounted() {

  }
}
</script>
<style lang="scss">

.info-con {
    @media(max-width: 1200px) {
        display: none!important;
    }
}

.mobile {
    display: none;
    @media(max-width: 1200px) {
        display: block!important;
    } 
}
.margin-right-small {
    margin-right: 10px;
}
.stream-info {
    margin-top: 42px;
    padding-right: 20px;
    margin-bottom: 50px;
}

.firstlabel {
    margin-top: 5px!important;
}
.cc-holder{
    background-color: #e8e8e8;
    color: #555;
    padding: 9px;
    margin-top: 10px;
    border-radius: 2px;
    p {
        float: right;
        margin-top: 2px;
        margin-right: 10px;
        font-size: 14px;
    }
}

small.warn {
    color: #dc3545;
    font-weight: 600;
    font-size: 13px;
}

.btn-primary {
    padding-top: 50px;
    background-color: #4f1fe1;
    border: none;
}

html, body {
    font-family: 'Open Sans', sans-serif;
}

td {
    font-size: 13px;
    .key {width: 100px;}
}

.enlarge-icn {
    font-size: 32px;
}

.center-load {
    text-align: center;
}
.form-widget {
    &.load {
   
        
        .load-info {
            text-align: center;
            margin-top: 20px;
            font-weight: 600;
            color: #120e18;
           
        }
         p {
                font-size: 14px;
                color: #120e18;
            }
    }
    margin-top: 50px;
    width: 370px;
    height: 510px;
    background-color: white;
    border-radius: 5px;
    // -webkit-box-shadow: 9px 18px 31px -3px rgba(199,191,199,0.5);
    // -moz-box-shadow: 9px 18px 31px -3px rgba(199,191,199,0.5);
    // box-shadow: 9px 18px 31px -3px rgba(199,191,199,0.5);
    padding: 24px;
    padding-top: 20px;


    label {
        margin-top: 27px;
        font-weight: 500;
        color: #120e18;
    }
    input {
        height: 40px;
        margin-top: 5px;
        border-radius: 2px;
    }
    .btn {
        margin-top: 20px;
        width: 100%;
        padding-top: 12px;
        height: 50px;
        &.dis {
            background-color: #8e8f8e;
            border-color: #8e8f8e;
        }
    }
}
.stream-widget {
    width: 360px;
    margin-top: 50px;
    background-color: yellow;
    height: 510px;
    border-radius: 6px;
    background: #8E2DE2;  /* fallback for old browsers */
    background: -webkit-linear-gradient(to right, #4A00E0, #8E2DE2);  /* Chrome 10-25, Safari 5.1-6 */
    background: linear-gradient(to right, #4A00E0, #8E2DE2); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
    box-shadow: 0px 4px 4px rgba(0, 0, 0, 0.25);
    position: relative;

    &.await {
        height: 510px;

        .lds-ripple {
            margin-top: 10px;
        }
        .total {
            position: absolute;
            padding-top: 280px;
            font-size: 13px;
            font-weight: 600;
        }

        .currency {
            padding-top: 90px;
        }

        .info {
            padding: 20px;
            font-weight: 400;
        }
    }
    .info {
        position: absolute;
        padding: 50px;
        padding-top: 40px;
        color: white;
        text-align: center;
        font-size: 15px;
    }

    .currency {
        position: absolute;
        padding-top: 150px;
        color: white; 
        font-size: 45px; 
        text-align: center;
        width: 100%;
        .den {
            font-weight: 600;
            font-size: 20px;
            margin-bottom: 0;
        }
        p {
            margin-bottom: 0;
            text-align: center;
        }
        span {
            color: #cfd0cf;
        }
    }

    .prog {
        position: absolute;
        padding-top: 260px;
        padding-left: 35px;
        padding-right: 35px;
        width: 100%;
        .progress-bar {
        background-color: #00FF85;
        }
        .progress {
            background-color: black;
            
        }
        .start {
            position: relative;
            left: 0;
            top: 10px;
            color: white;
            font-size: 12px;
        }

        .end {
            position: relative;
            left: calc(100% - 40px); 
            width: 100%;
            top: -6px;
            color: white;
            font-size: 12px;
        }
    }
    .total {
        position: absolute;
        padding-top: 340px;
        font-size: 22px;
        color: white;
        width: 100%;
        text-align: center;
    }

    .payment-info {
        position: absolute;
        padding-top: 470px;
        font-size: 12px;
        color: white;
        width: 100%;
        text-align: center; 
    }
}

.align-t-left {
    text-align: left;
    padding-left: 50px;
    @media(max-width: 1200px) {
    margin-right: 150px!important;
    padding-left: 20px!important;
    }
    .first {
        @media(max-width: 880px) {
            margin-top: 0px!important;
        }
    }
    h4 {
        @media(max-width: 880px) {
            margin-top: 30px;
        }
    }
}

.lds-ripple {
  display: inline-block;
  position: relative;
  width: 80px;
  height: 80px;
}
.lds-ripple div {
  position: absolute;
  border: 4px solid #fff;
  opacity: 1;
  border-radius: 50%;
  animation: lds-ripple 1s cubic-bezier(0, 0.2, 0.8, 1) infinite;
}
.lds-ripple div:nth-child(2) {
  animation-delay: -0.5s;
}
@keyframes lds-ripple {
  0% {
    top: 36px;
    left: 36px;
    width: 0;
    height: 0;
    opacity: 0;
  }
  4.9% {
    top: 36px;
    left: 36px;
    width: 0;
    height: 0;
    opacity: 0;
  }
  5% {
    top: 36px;
    left: 36px;
    width: 0;
    height: 0;
    opacity: 1;
  }
  100% {
    top: 0px;
    left: 0px;
    width: 72px;
    height: 72px;
    opacity: 0;
  }
}

.lds-ring {
  display: inline-block;
  position: relative;
  width: 80px;
  height: 80px;
}
.lds-ring div {
  box-sizing: border-box;
  display: block;
  position: absolute;
  width: 64px;
  height: 64px;
  margin: 8px;
  border: 8px solid #fff;
  border-radius: 50%;
  animation: lds-ring 1.2s cubic-bezier(0.5, 0, 0.5, 1) infinite;
  border-color: black transparent transparent transparent;
}
.lds-ring div:nth-child(1) {
  animation-delay: -0.45s;
}
.lds-ring div:nth-child(2) {
  animation-delay: -0.3s;
}
.lds-ring div:nth-child(3) {
  animation-delay: -0.15s;
}
@keyframes lds-ring {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

</style>