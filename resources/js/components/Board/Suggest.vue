<template>
    <div class="row col-md-9 mt-5">
        <div v-if="success" class="alert alert-success alert-dismissible fade show" role="alert">
            <strong>Thank you!</strong> Your suggestion has been received for processing.
            <button type="button" class="close" data-dismiss="alert" aria-label="Close">
                <span aria-hidden="true">&times;</span>
            </button>
        </div>
        <p class="text-left float-left">
            Help us populate this company's section 
            <a @click="show = !show" style="cursor: pointer; color: blue;"> by suggesting a board member </a> 
            that may be missing on this page.
        </p>
        <div class="card col-md-12 py-4 d-flex flex-row" v-if="show">
            <div class="col-md-3">
                <label for="picture">Profile Picture *</label>
                <div class="picture-area border-info">
                    <input type="file" accept="image/*" id="avatar" v-if="uploadImage" @change="preview" class="form-control">
                    <img :src="previewAvatar" alt="" v-if="!uploadImage">
                </div>
                <button class="mt-2 btn btn-danger btn-sm" v-if="!uploadImage" @click="changeImage">Change</button>

            </div>
            <div class="col-md-9">
                <form method="POST" @submit.prevent="submit" enctype="form-data/multipart">
                <span class="text-muted text-right w-full">Required fields are marked (*)</span>
                <!-- Company information -->
                <div class="row pb-4">
                    <div class="col">
                        <label for="name">Full Name *</label>
                        <input type="text" name="name" required class="form-control" id="name" placeholder="First name">
                    </div>
                    <div class="col">
                        <label for="position">Position *</label>
                        <input type="text" name="position" required class="form-control" placeholder="Position" id="position">
                    </div>
                </div>
                <!-- Socials -->
                <div class="row pb-4">
                    <div class="col">
                        <label for="linkedin">LinkedIn</label>
                        <input type="text" name="linkedin" class="form-control" placeholder="LinkedIn" id="linkedin">
                    </div>
                    <div class="col">
                        <label for="email">Email *</label>
                        <input type="email" name="email" class="form-control" id="email" placeholder="Email address">
                    </div>
                </div>
                <!-- More socials -->
                <div class="row pb-4">
                    <div class="col">
                        <label for="twitter">Twitter</label>
                        <input type="text" name="twitter" class="form-control" placeholder="Twitter URL" id="twitter">
                    </div>
                    <div class="col">
                        <label for="website">Website</label>
                        <input type="url" name="website" class="form-control" id="website" placeholder="Website URL">
                    </div>
                    <div class="col">
                        <label for="facebook">Facebook</label>
                        <input type="url" name="facebook" class="form-control" id="facebook" placeholder="Facebook URL">
                    </div>
                </div>
                    <!-- Submit -->
                    <button type="submit" :disabled="busy" class="form-control">
                        Submit
                        <div class="spinner-border spinner-border-sm" v-if="busy" role="status">
                            <span class="sr-only">Loading...</span>
                        </div>
                    </button>
                </form>
            </div>
        </div>
    </div>
</template>

<script>

export default {
    name: "Suggest",
    
    props:{
        company:{
            type: String,
            required: true,
        }
    },

    data() {
        return {
            fileReader: new FileReader(),
            busy: false,
            uploadImage: true,
            avatar: '',
            previewAvatar: '',
            success: false,
            show: false,
        }
    },

    methods: {
        preview(event){
            this.uploadImage = false;

            this.fileReader.onload = () => {
                this.previewAvatar = this.fileReader.result;
            }
            //set the raw file to avatar for form upload
            this.avatar = event.target.files[0];

            //stream the file preview 
            this.fileReader.readAsDataURL(event.target.files[0]);
        },

        changeImage(){
            this.uploadImage = true;
            this.previewAvatar = null;
        },

        submit(){
            //busyyyyyyyyy
            this.busy = true;

            //target the form element
            const person = document.querySelector("form");
            //FormData magically extracts all form elements 
            const form = new FormData(person);

            //append extra values
            form.append('avatar', this.avatar);
            //set the company id from the prop
            form.append('company_id', this.company); 

            axios.post('/api/companies/board/suggest', form).then((result) => {
                //done with processing
                this.busy = false;

                console.log(result);

                //remove uploaded image preview
                this.previewAvatar = null;
                this.uploadImage = true; //hide cancel button
                
                //empty all form values
                person.reset();

                //show success message
                this.success = true;
            }).catch((err) => {
                console.log(err);
            });
        }
    },
}
</script>

<style scoped>
    .picture-area{
        border: 1px solid;
        display: flex;
        justify-content: center;
        overflow: hidden;
        height: 160px;
        max-height: 160px;
        align-items: center;
    }

    .picture-area img{
        height: 100%;
        width: auto;
    }
</style>