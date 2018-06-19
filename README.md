
FOR PRIVACY AND CODE PROTECTING REASONS THIS IS A SIMPLIFIED VERSION OF CHANGES AND NEW FEATURES

TASK DATE: 13.06.2108 - FINISHED: 19.06.2018

TASK LEVEL: ADVANCED 

TASK SHORT DESCRIPTION: [
							366 - Tag photos by year and name
							1164 - Add categories to Photos/Gallery - like resources (see example there) changes on public site as well - drop down list of categories0
							1578 - Ability to limit access of photo galleries to certain user groups by adding a 
								   User Group tag field in the photo gallery form - Edit gallery - put categories dropdown here, and able to tag user_groups which can be seen the gallery
						]

GITHUB REPOSITORY CODE: feature/task-366-1164-1578-photo-gallery-categories-and-tagging

CHANGES

	NEW FILES	
		.....\views\content\galleries.php
		.....\views\content\gallery_categories.php
		.....\views\content\gallery_categories_form.php
		.....\views\content\tables\gallery_categories.php
		.....\controllers\content_gallery_categories.php
		.....\views\content\partials\tag_snippet.php
		.....\english\gallery_categories_lang.php
		.....\models\gallery_categories_m.php
		
		
	IN FILES: 
	
		.....\models\tags_m.php
		
			ADDED two functions 
			
				public function setPersonTypesForGalleries($galleryId, $personTypeIds = array() )
				{
					$this->db->where('gallery_id', $galleryId)->where('person_type_id IS NOT NULL', NULL, FALSE)->delete($this->_table);

					.....
				}//END function setPersonTypesForGalleries
				
				
				public function getPersonTypesForGallery($id)
				{
					return $this->db
						->select('pt.*')
						->from('tags t')
						.....
				}//END function getPersonTypesForGallery
		
	
	
	
		.....\views\gallery_layout.php
	
			CHANGED CODE
				
				FROM: 
					<div class="col-md-3 col-sm-12">
				TO: 
					<div class="col-md-3 col-sm-3 photo-gallery-category" data-category-id="<?=$gallery->category_id;?>" data-parent-category-id="<?=$gallery->parent_id;?>">
	
	
	
		.....news\models\photo_gallery_m.php
	
			ADDED new function
	
				public function getAllGalleries($orderField = '', $orderDirection = '', $params = array())
				{
					if ($orderField != '' and $orderDirection != '') {
						$this->db->order_by($orderField, $orderDirection);
					}
					
					if (isset($params['offset']) and isset($params['limit'])) {
						$this->db->limit($params['limit'], $params['offset']);
					}
					
					if (isset($params['where']) and is_array($params['where'])) {
						$this->db->where($params['where']);
					}
					
					.....
					
					return $resultGalleries;
				}//END function getAllGalleries
	
	
	
	
		.....\controllers\galleries.php
		
			ADDED CODE:
			
				function view_galleries()
				
					NOTE: gallery_categories_m is loaded in __construct function
					....
					->set('galleryMainCategories', $this->gallery_categories_m->getMainCategories())
					->set('gallerySubCategories', $this->gallery_categories_m->getSubCategories())
					....
		
	
			CHANGED CODE 
			
				inside function view_galleries()
				
					FROM: 
						$galleries = $this->photo_gallery_m->limit($pagination['limit'], $pagination['offset'])->order_by('order_id', 'desc')->get_many_by('published', true);
					TO:
						$galleries = $this->photo_gallery_m->getAllGalleries(
							$orderField = 'order_id', 
							$orderDirection = 'desc', 
							$settings = array(
								'filter_by_person_types' => true, 
								'where' => array('published' => true), 
								'limit' => $pagination['limit'], 
								'offset' => $pagination['offset']
							)
						);
					
					
	
	
		.....\views\gallery_grid.php
		
			ADDED CODE: 
			
				Inside div <div id="filter-stage">
	
					<div class="bottom-line border-top border-box top-stories top-news">
						<div class="row gallery-filters">
							<div class="col-md-3 col-sm-12">
								<p id="filter_label">Filter the list</p>
							</div>
							<div class="col-md-4 col-sm-12">
								<select id="gallery_main_categories">
									<option value="0" selected>Select a category</option>
									<?php foreach($galleryMainCategories as $category):?>
										<option value="<?=$category->id;?>"><?=$category->title;?></option>
									<?php endforeach;?>
								</select>
							</div>
							<div class="col-md-4 col-sm-12">
								<select id="gallery_sub_categories">
									<option data-parent-id="0" value="0" selected>Select a sub-category</option>
									<?php foreach($gallerySubCategories as $category):?>
										<option data-parent-id="<?=$category->parent_id;?>" value="<?=$category->id;?>" style="display:none"><?=$category->title;?></option>
									<?php endforeach;?>
								</select>
							</div>
						</div>
					</div>
					
					
	
		.....\css\gallery.css
		
			ADDED CODE: 
			
				/************* FILTERING GALLERIES ***********/

				select#gallery_main_categories, 
				select#gallery_sub_categories {
					border: 1px solid #ccc;
					box-shadow: none;
					background: #fff;
					color: #555;
					padding: 12px;
					margin-top: 5px;
					border-radius: 0;
					/*for firefox*/
					-moz-appearance: none;
					/*for chrome*/
					-webkit-appearance:none;
				}
				select#gallery_sub_categories {
					display: none;
				}
				select#gallery_main_categories::-ms-expand, 
				select#gallery_sub_categories::-ms-expand {
					/*for IE10*/
					display: none;
				}
				.gallery-filters #filter_label {
					font-weight: bold;
					margin-top: 10px
				}
		
		
		
	
		.....\js\gallery.js
	
			CHANGED CODE:
			
				//create new gallery popup
				$(document).on('click', "#btnCreateNewGallery", function () {
					$('.modal-title.create-gallery').css('display', 'block');
					$('.modal-title.edit-gallery').css('display', 'none');
					$('#btnCreateGallery').css('display', 'inline');
					$('#btnEditGallery').css('display', 'none');
					 $('#new-gallery-form').attr('action', BASE_URL + 'admin-portal/content/galleries/create');
					$('#myAddGalleryModal .chosen-container').each(function(i, ele){
						if ($(ele).width() == 0) {
							$(ele).css('width', '236px');
							$(ele).find('.chosen-drop').css('width', '234px');
							$(ele).find('.chosen-search input').css('width', '200px');
							$(ele).find('.chosen-field input').css('width', '225px');
						}
					});
					$('#myAddGalleryModal').modal('show');

					// checks Create Gallery title field has a value - for browsers where HTML5 'required' doesn't work - like Safari
					$(document).on('click', "#btnCreateGallery", function () {  
					if ( ! $('#add_gallery_title').val() ) {
						  alert( 'Please enter a title.' );
						  return false;
					  }
					});
				});
	
			CHANGED CODE II:
			
				FROM: 
					 $(document).on('click', '.edit-gallery', function(){
						var gallery_id = $(this).data('gallery-id');
						$('#edit_gallery_title').val('');
						$('#edit_gallery_subtitle').val('');
						$('#edit-gallery-form').attr('action', BASE_URL + 'admin-portal/content/galleries/edit/' + gallery_id);


						$.ajax({
							type: "POST",
							url: 'galleries/edit/' + gallery_id,
							success:function (response){
								if (response.status == 'success') {
									$('#edit_gallery_title').val(response.title);
									$('#edit_gallery_subtitle').val(response.subtitle);
								.....
								}
								else if (response.status == 'error') {
									// show our errors
									errors.css('display', 'block');
								}
								else
								{
									console.log('filter '+response.status);
								}
							}
						});

					});
				
				TO: 
					$(document).on('click', '.edit-gallery', function(){
						var gallery_id = $(this).data('gallery-id');
						$('#add_gallery_title').val('');
						$('#f_gallery_subtitle').val('');
						$('#person_type_tag').val('');
						$("#person_type_tag_tagsinput span.tag").remove();
						.....
						
						$.ajax({
							type: "POST",
							url: 'galleries/edit/' + gallery_id,
							success:function (response){
								if (response.status == 'success') {
									$('#add_gallery_title').val(response.title);
									$('#f_gallery_subtitle').val(response.subtitle);
								.....
								}
								else if (response.status == 'error') {
									// show our errors
									errors.css('display', 'block');
								}
								else
								{
									console.log('filter '+response.status);
								}
							}
						});

					});
	
			ADDED functions: 
			
				function updateSubSelectbox(mainSelected) 
				{
					$("#gallery_sub_categories option").each(function(){
						.....
					});
				}//END function updateSubSelectbox
				
				
				function filterGalleryList(mainSelected, subSelected) 
				{
					//Showing galleries from the subcategory only
					if(subSelected != "0") {
						$(".photo-gallery-category").each(function(){
							.....
						});
					}
					//Showing galleries from the main category and as well from all its subcategories
					else if(subSelected == "0" && mainSelected != "0") {
						.....
					}
					//Showing all resources from all categories
					else {
						$(".photo-gallery-category").each(function(){
							$(this).show();
						});
					}
				}//END function filterGalleryList
				
	
			function removePersonTypeTag(personTypeValue, domElement)
			{
				var personTypes = $('#person_type_tag').val();
				personTypes = personTypes.replace("," + personTypeValue, "");
				.....
				domElement.parent('span').remove();
			}

				
		
		.....\controllers\content.php
		
			CHANGED CODE: 
			
				Inside function galleries()
				
					FROM: 
						$galleries = $this->photo_gallery_m->order_by('order_id', 'DESC')->get_all();
					TO:
						$galleries = $this->photo_gallery_m->getAllGalleries('order_id', 'DESC');
		
			ADDED CODE: 
			
				Inside function galleries()
					....
					$this->load->model('.....gallery_categories_m');
					$this->lang->load('.....gallery_categories');
					....
					
			CHANGED FUNCTION create_gallery: 	

				public function create_gallery()
				{
					$this->load->model('tags/tags_m');
					
					$this->form_validation->set_rules($this->galleriesValidationRules);

					if ($this->form_validation->run()) {
						.....
						
						if ($gallery_id) {
							$personTypeIds = array();
							$presonTypeTags = ( $this->input->post('person_type_tag') != "") ? explode(',', $this->input->post('person_type_tag')) : array();
							foreach ($presonTypeTags as $t) {
								$personTypeIds[] = array_pop(explode(':', $t));
							}	
							$this->tags_m->setPersonTypesForGalleries($gallery_id, $personTypeIds);

							redirect('...../galleries/edit/' . $gallery_id);
						}
					}
				}
								
		
			CHANGED FUNCTION edit_gallery: 
			
				public function edit_gallery($id)
				{
					Asset::enable('jquery.template');
					Asset::enable('jquery.file.uploader');

					$gallery = $this->photo_gallery_m->get($id);
					
					$this->form_validation->set_rules($this->galleriesValidationRules);

					if ($this->form_validation->run()) {
						.....
						rename(UPLOAD_PATH . 'galleries/' . $gallery->slug, UPLOAD_PATH . 'galleries/' . $slug);

						$data['slug'] = $slug;

						if ($this->photo_gallery_m->update($id, $data)) {
							redirect('...../galleries');
						}

					}

					if ($this->input->is_ajax_request()) {
						$status = 'error';
						if (isset($gallery)) {
							.....
						} else {
							$response = array(
								'status' => $status
							);
						}

						$this->template->build_json($response);

					} else {
						$this->template
							->set('gallery', $gallery)
							->set('clubs_name', $this->clubs_name)
							.....
					}
				}

					
	
		
		.....\views\content\tables\galleries.php
		
			ADDED CODE: 
			
				.....
				<th>Category</th>
				.....
				<td><?=$gallery->category_title?></td>
				.....
				
		
		
		
		.....\js\content.js
		
			ADDED CODE:
			
				/* deleting gallery category after confirmation */
				$(document).on('click', '#confirm-delete-gallery-category', function() 
				{
					var type = $('#myModal').data('type');

					if ( type == 'single' ) {
						var id = deleteGalleryCategories$('#myModal').data('id');
						(id);
					}
					else if ( type == 'multiple' ) {
						deleteGalleryCategories(0);
					}
				});	
	
			ADDED NEW FUNCTION 
			
				function deleteGalleryCategories(id) 
				{
					//Disable buttons on modal
					COMMON.disableButton('#confirm-delete-gallery-category');
					COMMON.disableButton('#cancel-delete-gallery-category');
					
					//Init variables
					var errors = $('#admin-notices');
					var ids = [];
							
					//Getting deletable records' ids
					.....
					
					//Deleting records
					AJAX.call('...../categories/delete', {'ids' : ids}, function(r) {
						response = $.parseJSON(r);
						if (response.status == 'success') {
							location.reload();
						}
						else {
							.....
						}
					});
				}//END function deleteGalleryCategories
	
	
	
	
		.....\views\content\galleries.php
			
			TAKEN OUT:
				<div class="modal fade" id="myEditGalleryModal" tabindex="-1" role="dialog" aria-labelledby="myEditGalleryModalLabel" aria-hidden="true">
					<div class="modal-dialog">
						<div class="modal-content">
							<div class="modal-header">
								<button type="button" class="close" data-dismiss="modal" aria-hidden="true">&times;</button>
								<h4 class="modal-title">Edit gallery</h4>
							</div>
							<?php echo form_open(site_url(''), array('class'=>'form_inputs', 'id' => 'edit-gallery-form'));?>
							<div class="modal-body">
								<fieldset>
									<ul>
										<li>
											<label for="edit_gallery_title">Title</label>
											<div class="input">
												<?=form_input('title', '', 'class="form-control" style="width:236px;" id="edit_gallery_title" maxlength="150" required ') ?>
											</div>
										</li>
									.....
									</ul>
								</fieldset>
							</div>
							<div class="modal-footer">
								<button type="button" class="btn btn-default" data-dismiss="modal">Cancel</button>
								<button id="btnEditGallery" class="btn btn-primary" type="submit">Save</button>
							</div>
							<?php echo form_close(); ?>
						</div><!-- /.modal-content -->
				  <!--  </div> /.modal-dialog -->
				<!-- </div> /.modal -->			
				
	
	
			ADDED CODE: (end of the file)
			
				<script>
				$(function(){
					$('#gallery_category_id_chosen').css({"min-width" : "210px", "width" : "236px"});
					$('#person_type_tag_tagsinput').css({"min-width" : "210px", "width" : "236px"});
				});
				</script>
	
			CHANGED MODAL:

				<!-- Two modals merged into one -->
				<div class="modal fade" id="myAddGalleryModal" tabindex="-1" role="dialog" aria-labelledby="myAddGalleryModalLabel" aria-hidden="true">
					<div class="modal-dialog">
						<div class="modal-content">
							<div class="modal-header">
								<button type="button" class="close" data-dismiss="modal" aria-hidden="true">&times;</button>
								<h4 class="modal-title create-gallery">Create a new gallery</h4>
								<h4 class="modal-title edit-gallery">Edit gallery</h4>
							</div>
							<?php echo form_open(site_url('...../galleries/create'), array('class'=>'form_inputs', 'id' => 'new-gallery-form'));?>
							<div class="modal-body">
								<fieldset>
									<ul>
										<li>
											<label for="add_gallery_title">Title</label>
											<div class="input">
												<?=form_input('title', '', 'class="form-control" style="width:236px;" id="add_gallery_title" maxlength="150" required') ?>
											</div>
										</li>

										.....
									</ul>
								</fieldset>
							</div>
							<div class="modal-footer">
								<button type="button" class="btn btn-default" data-dismiss="modal">Cancel</button>
								.....
							</div>
							<?php echo form_close(); ?>
						</div><!-- /.modal-content -->
					</div><!-- /.modal-dialog -->
				</div><!-- /.modal -->

				
			CHANGED CODE: 
			
				FROM: 
					<section class="title">
						<h4>Photo Galleries</h4>
						<a id="btnCreateNewGallery" class="btn btn-primary create pull-right"><span class="glyphicon glyphicon-plus"></span> Create Gallery</a>                
					</section>
					<div class="content full-col">
								<?php
								if(!$enabled_galleries) {
									$btn='btn-success';
									$val=1;
									$btn_val='Enable';
									$text_val='disabled';
								}
								else {
									$btn='btn-warning';
									$val=0;
									$btn_val='Disable';
									$text_val='enabled';
								}
								?>
						<div style="margin-bottom: 1em;">
							<p>Photos menu tab is currently <?=$text_val?></p>
							<button class="btn <?=$btn?>" id="galleries_toggle" value="<?=$val?>"><?=$btn_val?></button>
						</div>
					</div>
			
				TO: 
					<section class="title">
						<h4>Photo Galleries</h4>
						<a href="<? echo base_url()?>...../galleries/categories" class="btn btn-primary pull-right">Categories</a>	
						<a id="btnCreateNewGallery" class="btn btn-primary create pull-right"><span class="glyphicon glyphicon-plus"></span> Create Gallery</a>
					</section>
					<div class="content full-col">
						<?php
							if(!$enabled_galleries) {
								$btnClass = 'btn-success';
								$btnValue=1;
								$btnCaption='Enable';
								$text='disabled';
							}
							else {
								$btnClass='btn-warning';
								$btnValue=0;
								$btnCaption='Disable';
								$text='enabled';
							}
						?>
						<div style="margin-bottom: 1em;">
							<p>Photos menu tab is currently <?=$text?></p>
							<button class="btn <?=$btnClass?>" id="galleries_toggle" value="<?=$btnValue?>"><?=$btnCaption?></button>
						</div>
					</div>
	
	
	
	
		.....\details.php
		
			ADDED CODE: 
			
				Inside function upgrade
				
					//Creating new tables for managing gallery categories
					if (version_compare($old_version, '2.2.3', 'lt')) {
						$this->_addPhotoGalleryCategoriesTable();
						$this->_addFieldPhotoGalleryTable(array('category_id'));
						$this->_addFieldTagsTable(array('gallery_id'));
					}
					
				Inside function install 
				
					$this->_addPhotoGalleryCategoriesTable();
					$this->_addFieldPhotoGalleryTable(array('category_id'));
					$this->_addFieldTagsTable(array('gallery_id'));
				
				added new functions
				
					private function _addPhotoGalleryCategoriesTable() 
					{
						$this->db->query("
							CREATE TABLE IF NOT EXISTS `....._gallery_categories` (
								`id` int(11) unsigned NOT NULL AUTO_INCREMENT,
								`parent_id` int(11) unsigned NOT NULL DEFAULT '0',
								.....
							) ENGINE=InnoDB  DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci AUTO_INCREMENT=1 ;
						");				
					}//END function _addPhotoGalleryCategoriesTable

					
 
					public function _addFieldPhotoGalleryTable( $fields = array() ) 
					{
						if ( count($fields) > 0 ) {
							$table = $this->db->dbprefix('photo_gallery');
							
							# ADD category_id column to table
							if ( in_array('category_id', $fields) ) {
								if ( ! $this->db->field_exists('category_id', $table)) {
									.....
								}
							}#END in_array('category_id', $fields)
						}
					} //END function _addFieldPhotoGalleryTable
					
					
					
					public function _addFieldTagsTable( $fields = array() ) 
					{
						if ( count($fields) > 0 ) {
							$table = $this->db->dbprefix('tags');
							
							# ADD category_id column to table
							if ( in_array('gallery_id', $fields) ) {
								.....
							}#END in_array('gallery_id', $fields)
						}
					} //END function _addFieldTagsTable	
