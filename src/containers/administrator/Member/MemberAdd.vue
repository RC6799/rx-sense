<template>
  <main>
    <v-toolbar dark>
      <v-btn icon @click="back" dark>
        <v-icon>arrow_back</v-icon>
      </v-btn>
      <v-toolbar-title class="white--text">Add Member / {{convertString(active)}} </v-toolbar-title>
      <v-spacer></v-spacer>
    </v-toolbar>
    <v-form ref="form">
    <v-container fluid>
      <v-layout row wrap>
        <v-tabs v-model="active">
          <v-tabs-bar class="transparent">
            <v-tabs-slider class="active-line"></v-tabs-slider>
            <v-tabs-item
              v-for="tab in tabs"
              :key="tab.name"
              :href="tab.path"
              ripple
            >
              {{ tab.name }}
            </v-tabs-item>
          </v-tabs-bar>
          <v-tabs-items>
            <v-tabs-content key="member" id="member">
              <member type="add" :data="info"></member>
            </v-tabs-content>
            <v-tabs-content key="assign" id="assign_to_plans">
              <member-assign type="add" :data="[]"></member-assign>
            </v-tabs-content>
            <v-tabs-content key="drugList" id="drug_list">
              <member-drug-list type="add" :data="info"></member-drug-list>
            </v-tabs-content>
            <v-tabs-content key="personalInfo" id="personal_info">
              <member-personal-info type="add" :data="info"></member-personal-info>
            </v-tabs-content>
            <v-tabs-content key="benefits" id="benefits">
              <member-benefits type="add" :data="info"></member-benefits>
            </v-tabs-content>
            <v-tabs-content key="insurance" id="insurance">
              <member-insurance type="add" :data="info"></member-insurance>
            </v-tabs-content>
            <v-tabs-content key="address" id="address">
              <member-address type="add" :data="info"></member-address>
            </v-tabs-content>
            <v-tabs-content key="employer" id="employer">
              <member-employer type="add" :data="info"></member-employer>
            </v-tabs-content>
            <v-tabs-content key="comments" id="comments">
              <member-comments type="add" :data="info"></member-comments>
            </v-tabs-content>
          </v-tabs-items>
        </v-tabs>
        <v-layout>
          <v-card-text class="py-0 px-1">
            <v-divider class="mt-2"></v-divider>
            <small class="red--text">*indicates required field</small>
            <v-card-actions>
              <v-btn @click.native="clear" :disabled="isDisabledSave">Clear</v-btn>
              <v-spacer></v-spacer>
              <v-btn class="primary" @click.native="save" :disabled="isDisabledSave">create</v-btn>
            </v-card-actions>
          </v-card-text>
        </v-layout>
      </v-layout>
    </v-container>
    </v-form>
  </main>
</template>

<script>
  import memberServices from '../../../services/member/memberServices'
  import store from '../../../store'
  import Member from './Member/Member.vue'
  import MemberAssign from './Assign/Assign.vue'
  import MemberDrugList from "./DrugList/DrugList.vue";
  import MemberPersonalInfo from "./PersonalInfo/PersonalInfo.vue";
  import MemberBenefits from "./Benefits/Benefits.vue";
  import MemberInsurance from './Insurance/Insurance.vue'
  import MemberAddress from './Address/Address.vue'
  import MemberEmployer from "./Employer/Employer.vue";
  import MemberComments from "./Comments/Comments.vue";

  export default {
    name: 'member-add',
    data() {
      return {
        tabs: [],
        active: 'member',
        info: {},
        isDisabledSave: true,
        rules: {
          required: (value) => !!value || 'Required.',
          birthdayRules: (v) => v <= (new Date()).toLocaleDateString() || 'Must be less than current date'
        }
      }
    },
    components: {
      Member, MemberPersonalInfo, MemberInsurance, MemberEmployer, MemberDrugList,
      MemberComments, MemberBenefits, MemberAssign, MemberAddress
    },
    watch: {
      info: {
        handler: function (newData, oldData) {
          if (newData && oldData && newData === oldData)
            this.isDisabledSave = false
        }, deep: true
      }
    },
    mounted() {
      console.log('----- Administrator / Member Add mounted -----')
      this.setValues()
      this.info = {...this.$store.state.selectedMember}
    },
    methods: {
      convertString: function (text) {
        if (text) return text.replace(/_/g, ' ')
        else return ''
      },
      setValues: function () {
        const formType = this.$store.state.memberFormType
        if (formType === 0) {
          this.tabs = [
            {name: 'Member', path: 'member'},
            {name: 'Assign To Plans', path: 'assign_to_plans'},
            {name: 'Drug List', path: 'drug_list'},
            {name: 'Personal Info', path: 'personal_info'},
            {name: 'Benefits', path: 'benefits'},
            {name: 'Insurance', path: 'insurance'},
            {name: 'Address', path: 'address'},
            {name: 'Employer', path: 'employer'},
            {name: 'Comments', path: 'comments'}
          ]
        } else {
          this.tabs = [
            {name: 'Member', path: 'member'},
            {name: 'Assign To Plans', path: 'assign_to_plans'},
            {name: 'Drug List', path: 'drug_list'},
            {name: 'Personal Info', path: 'personal_info'},
            {name: 'Benefits', path: 'benefits'},
            {name: 'Address', path: 'address'},
            {name: 'Comments', path: 'comments'}
          ]
        }
      },
      back: function () {
        store.commit('UPDATE_STATUS', true)
        store.commit('CLEAR', 'selectedMember')
        this.$router.push({name: 'member'})
      },
      save() {
        const that = this
        if (this.$refs.form.validate()) {
          if (this.info.FirstName === ''
            || this.info.LastName === ''
            || this.info.PersonCode === ''
            || this.info.MemberId === ''
            || this.info.DateOfBirth > (new Date()).toLocaleDateString()
          ) {
            this.active = 'member'
          } else {
            memberServices.addMember(this.info, this.$store.state.user).then(function (newMem) {
              that.isDisabledSave = true
              store.commit('UPDATE_STATUS', true)
              window.Vue.$emit('snackbar', 'success', 'Member  Data Created!')
              store.commit('CLEAR', 'selectedMember')
              store.commit('CLEAR', 'selectedPlanMembers')
              store.commit('CLEAR', 'selectedMemberDrugs')
              that.$router.push({name: 'member'})
            })
          }
        }
      },
      clear: function () {
        const d = new Date()
        d.setDate(d.getDate() - 1)
        this.info = {
          Id: 0,
          MemberId: '',
          PersonCode: '',
          IsActive: true,
          Pcc: this.$store.state.pcn.pcc,
          LastName: '',
          FirstName: '',
          MiddleName: '',
          MailingAddress: {
            County: '',
            Street2: '',
            Street1: '',
            City: '',
            State: '',
            PostalCode: ''
          },
          HomePhoneNumber: '',
          WorkPhoneNumber: '',
          WorkPhoneNumberExt: '',
          CellPhoneNumber: '',
          HomeFaxNumber: '',
          WorkFaxNumber: '',
          PagerNumber: '',
          EmailAddress: '',
          DateOfBirth: (new Date(d)).toLocaleDateString(),
          SocialSecurityNumber: '',
          Gender: 0,
          Relationship: 0,
          PrimaryMemberId: '',
          PrimaryPersonCode: '',
          MaritalStatus: 0,
          DrugTemplateId: 0,
          DependentCount: 0,
          IsStudent: false,
          IsDisabled: false,
          IsSmoker: false,
          IsPregnant: false,
          IsDiabetic: false,
          HasAllergies: false,
          EmployeeId: '',
          EmployerName: '',
          EmployerAddress: {
            Street1: '',
            City: '',
            State: '',
            PostalCode: ''
          },
          EmployerContactName: '',
          Occupation: '',
          CardholderId: '',
          CardholderLastName: '',
          CardholderFirstName: '',
          CardholderDateOfBirth: (new Date(d)).toLocaleDateString(),
          CardholderPlanId: '',
          InsuranceCompanyName: '',
          InsuranceAddress: {
            Street1: '',
            City: '',
            State: '',
            PostalCode: ''
          },
          InsurancePhoneNumber: '',
          AlternateMemberId: '',
          HasPrimaryCoverage: false,
          Comment: '',
          SubPlanId: null,
          PlanMembers: [],
          MemberDrugs: [],
          Allergy: {},
          Diagnosis: {},
          OtherInsurances: [],
          PcOrg: null,
          CreatedBy: '',
          CreatedDate: '',
          UpdatedBy: ''
        }

        store.commit('UPDATE_SELECTED_MEMBER', this.info)
        store.commit('UPDATE_SELECTED_PLAN_MEMBER', [])
        this.isDisabledSave = true
      }
    }
  }
</script>
<style lang="stylus">
</style>
