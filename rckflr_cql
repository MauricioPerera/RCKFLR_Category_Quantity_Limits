<?php
/*
 * Plugin Name: RCKFLR Category Quantity Limits
 * Plugin URI: https://rckflr.party/
 * Description: This plugin adds minimum and maximum quantity fields to product categories and enforces these limits in the WooCommerce cart.
 * Version: 1.0
 * Author: Mauricio Perera
 * Author URI: https://www.linkedin.com/in/mauricioperera/
 * Donate link: https://www.buymeacoffee.com/rckflr
*/

class RCKFLRCategoryQuantityLimits {
    public function __construct() {
        add_action( 'product_cat_edit_form_fields', array( $this, 'rckflr_add_category_quantity_fields' ), 10, 2 );
        add_action( 'edited_product_cat', array( $this, 'rckflr_save_category_quantity_fields' ), 10, 2 );
        add_action( 'woocommerce_check_cart_items', array( $this, 'rckflr_update_cart_item_quantity' ) );
    }

    public function rckflr_add_category_quantity_fields( $term, $taxonomy ){
        // Get existing category meta data
        $min_quantity = get_term_meta( $term->term_id, 'category_min_qty', true );
        $max_quantity = get_term_meta( $term->term_id, 'category_max_qty', true );
        
        // Display Min/Max fields
        ?>
        <tr class="form-field">
            <th scope="row" valign="top"><label for="category_min_qty"><?php esc_html_e( 'Min Quantity', 'woocommerce' ); ?></label></th>
            <td>
                <input type="number" name="category_min_qty" id="category_min_qty" value="<?php echo $min_quantity; ?>" min="0" step="1">
            </td>
        </tr>
        <tr class="form-field">
            <th scope="row" valign="top"><label for="category_max_qty"><?php esc_html_e( 'Max Quantity', 'woocommerce' ); ?></label></th>
            <td>
                <input type="number" name="category_max_qty" id="category_max_qty" value="<?php echo $max_quantity; ?>" min="1" step="1">
            </td>
        </tr>
        <?php
    }

    public function rckflr_save_category_quantity_fields( $term_id, $taxonomy ){
        if( isset( $_POST['category_min_qty'] ) ){
            update_term_meta( $term_id, 'category_min_qty', absint( $_POST['category_min_qty'] ) );
        }
        
        if( isset($_POST['category_max_qty'] ) ){
            update_term_meta( $term_id, 'category_max_qty', absint( $_POST['category_max_qty'] ) );
        }
    }

    public function rckflr_update_cart_item_quantity(){
        // Get cart items and category quantities
        $cart_items = WC()->cart->get_cart();
        $category_quantities = array();
        
        // Loop through cart items and get relevant category quantities
        foreach( $cart_items as $cart_item ){
            $categories = wp_get_post_terms( $cart_item['product_id'], 'product_cat', array( 'fields' => 'ids' ) );
            
            foreach( $categories as $category ){
                if( ! isset( $category_quantities[ $category ] ) ){
                    $category_quantities[ $category ] = 0;
                }
                
                $category_quantities[ $category ] += $cart_item['quantity'];
            }
        }
        
        // Loop through category quantities and update if necessary
        foreach( $category_quantities as $category_id => $quantity ){
            $min_quantity = get_term_meta( $category_id, 'category_min_qty', true );
            $max_quantity = get_term_meta( $category_id, 'category_max_qty', true );
            
            if( $quantity < $min_quantity ){
                $category_name = get_term( $category_id )->name;
                wc_add_notice( sprintf( __( '%s - Minimum quantity is %s', 'woocommerce' ), $category_name, $min_quantity ), 'error' );
                
                foreach( $cart_items as $cart_item ){
                    if( in_array( $category_id, wp_get_post_terms( $cart_item['product_id'], 'product_cat', array( 'fields' => 'ids' ) ) ) ){
                        WC()->cart->set_quantity( $cart_item['key'], $min_quantity );
                    }
                }
            }
            elseif( $quantity > $max_quantity ){
                $category_name = get_term( $category_id )->name;
                wc_add_notice( sprintf( __( '%s - Maximum quantity is %s', 'woocommerce' ), $category_name, $max_quantity ), 'error' );
                
                foreach( $cart_items as $cart_item ){
                    if( in_array( $category_id, wp_get_post_terms( $cart_item['product_id'], 'product_cat', array( 'fields' => 'ids' ) ) ) ){
                        WC()->cart->set_quantity( $cart_item['key'], $max_quantity );
                    }
                }
            }
        }
    }
}

new RCKFLRCategoryQuantityLimits();

?>
