<template>
    <div>
        <add-bank-account v-if="addBank" v-on:done="addBank = false"></add-bank-account>
        <v-card flat>
            <v-card-title>
                <span class="headline">Business Profile</span>
            </v-card-title>
            <v-card-text>
                <v-layout class="mt-2" wrap>
                    <template v-if="pictureUrl">
                        <v-flex class="text-xs-center" xs12>
                            <img :src="pictureUrl">
                        </v-flex>
                        <v-flex class="text-xs-center" xs12>
                            <p v-if="uploading">Uploading...</p>
                            <a v-else @click="$refs.file.click()">Change Picture</a>
                        </v-flex>
                    </template>
                    <template v-else>
                        <p v-if="uploading">Uploading...</p>
                        <a v-else @click="$refs.file.click()">Upload logo</a>
                    </template>

                    <input @change="uploadLogo" ref="file" accept="image/*" class="hidden" type="file"/>



                    <v-flex xs12>
                        <v-text-field v-model="name" label="Business Name">
                        </v-text-field>
                    </v-flex>
                    <v-flex xs12>
                        <v-text-field v-model="address" label="Business Address">
                        </v-text-field>
                    </v-flex> <v-flex xs12>
                    <v-text-field v-model="phone" label="Business Phone Number">
                    </v-text-field>
                </v-flex> <v-flex xs12>
                    <v-text-field v-model="website" label="Business Website">
                    </v-text-field>
                </v-flex>
                    <v-flex xs12>
                        <v-text-field v-model="email" label="Business Email">
                        </v-text-field>
                    </v-flex>
                    <v-flex x12>
                        <v-text-field v-model="supportEmail" label="Support Email">
                        </v-text-field>
                    </v-flex>
                    <v-flex xs12 class="text-xs-center">
                        <v-btn @click="update" :loading="loading" color="success" flat>Save Changes</v-btn>
                    </v-flex>


                </v-layout>
            </v-card-text>
        </v-card>

        <v-card class="mt-3">
            <v-card-title>
                <span class="headline">Bank Account</span>
            </v-card-title>

            <v-card-text>
                <v-progress-circular indeterminate v-if="bankLoading">

                </v-progress-circular>

               <template v-else-if="!bank">
                   <h5>No bank has been added. Please add a bank below.</h5>
                   <v-btn @click="addBank = true" flat color="primary">Add Bank Account</v-btn>
               </template>
                <template v-else>
                    <p>{{bank.get('name')}}</p>
                    <p>{{bank.get('no')}}</p>
                    <p>{{bank.bankName}}</p>
                </template>
            </v-card-text>
        </v-card>
    </div>

</template>

<script>
import Parse from 'parse';
import moment from 'moment';
import Util from '@/util';
import AddBankAccount from '../AddBankAccount';
export default {
  name: 'business-settings',
  components: { AddBankAccount },
  data () {
    return {
      pictureUrl: '',
      name: '',
      address: '',
      phone: '',
      website: '',
      email: '',
      supportEmail: '',
      loading: false,
      uploading: false,
      business: null,
      bank: null,
      bankLoading: true,
      addBank: false,

    };
  },
  async mounted () {
    const business = await Util.getCurrentBusiness();
    this.business = business;
    this.pictureUrl = business.getLogoUrl();
    this.name = business.getName();
    this.address = business.getAddress();
    this.email = business.getEmail();
    this.supportEmail = business.getSupportEmail();
    this.website = business.getWebsite();
    this.phone = business.getPhone();

    this.bank = await business.getBankAccount();

    this.bankLoading = false;
  },
  methods: {
    async uploadLogo () {
      const file = this.$refs.file.files[0];
      const fileName = file.name;
      this.uploading = true;
      const parseFile = new Parse.File(fileName, file);

      try {
        await parseFile.save();
        const res = await Parse.Cloud.run('updateBusinessProfile', { picture: parseFile, businessId: this.business.id });
        this.pictureUrl = parseFile.url();
        Util.toast(res);
      } catch (e) {
        Util.toast(e.message, false);
      } finally {
        this.uploading = false;
      }

    },
    async update () {
      this.loading = true;
      try {
        const response = await Parse.Cloud.run('updateBusinessProfile',
          { businessId: this.business.id, name: this.name, address: this.address, phone: this.phone, email: this.email, supportEmail: this.supportEmail, website: this.website });
        Util.toast(response);
      } catch (e) {
        Util.toast(e.message, false);
      }
      finally {
        this.loading = false;
      }
    }
  },
};
</script>
